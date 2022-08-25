<template>
  <div class="main">
    <!-- <Login @login = "loginFunction" @signup = 'signUpFunction'/> -->
    <Chat
      :messages="messages"
      @send-message="sendMessage"
      @join-channel="setChannel"
    />
  </div>
</template>

<script>
import Chat from "~/components/Chat.vue";
import Login from "../components/Login.vue";


export default {
  name: "IndexPage",
  components: { Chat, Login },
  head() {
    return {
      
    };
  },
  data() {
    return {
      node: null,
      messages: [],
      message: "",
      peers: [],
      peer: null,
      peerCount: 0,
      channel: null,
      lastAlive: 0,
      lastPeer: 0,
      bootstraps: [],
      lastBootstrap: 0,
      id: null,
      channelJoined: false,
      chatPeers: [],
    };
  },
  methods: {
    async setChannel(channel) {
      this.channel = channel;
      console.log("Channel set to " + channel);
      await this.node.pubsub.subscribe(this.channel, this.recieveMessage);
      this.channelJoined = true;
    },
    async recieveMessage(msg){
      const data = JSON.parse(new TextDecoder().decode(msg.data));
      if(this.verifyMessage(data)){
        this.messages.push(data);
      }else{
        return
      }
    },
    async sendMessage(message) {
      const msg = await this.signMessage(message);
      await this.node.pubsub.publish(
        this.channel,
        new TextEncoder().encode(JSON.stringify(msg))
      );
    },
    async  verifyMessage(data) {
      console.log(data)
      const signerAddress = await ethers.utils.verifyMessage(
        data.message,
        data.signature
      );
      if(signerAddress == data.address){
        return true;
      }else{
        return false;
      }
    },
    async signMessage(message){
      if(!window.ethereum) {
        alert("Please enable metamask");
        return;
      }
      await window.ethereum.send("eth_requestAccounts");
      const provider = new ethers.providers.Web3Provider(window.ethereum);
      const signer = provider.getSigner();
      const signature =   await signer.signMessage(message);
      const address = await signer.getAddress();
      const signedMessage = {
        message: message,
        signature: signature,
        address: address,
        time: new Date().toLocaleTimeString(),
        messageId: Math.floor(Math.random() * 1000000),
        from: this.id
      }
      return signedMessage
    },
    async createNode() {
      //create a node to be passed into the chat component
      const node = await Ipfs.create({
        repo: "ipfs-" + Math.random(),
        relay: {
          enabled: true,
          hop: {
            enabled: true,
          },
        },
        config: {
          Addresses: {
            Swarm: [
              "/dns4/wrtc-star1.par.dwebops.pub/tcp/443/wss/p2p-webrtc-star",
              "/dns4/wrtc-star2.sjc.dwebops.pub/tcp/443/wss/p2p-webrtc-star",
              "/dns4/star.thedisco.zone/tcp/9090/wss/p2p-webrtc-star",
              "/dns6/star.thedisco.zone/tcp/9090/wss/p2p-webrtc-star",
              "/dns6/ipfs.thedisco.zone/tcp/4430/wss/p2p/12D3KooWChhhfGdB9GJy1GbhghAAKCUR99oCymMEVS4eUcEy67nt",
              "/dns4/ipfs.thedisco.zone/tcp/4430/wss/p2p/12D3KooWChhhfGdB9GJy1GbhghAAKCUR99oCymMEVS4eUcEy67nt",
            ],
          },
        },
      });
      console.log(node);
      this.id = await node.id();
      this.node = node;
      const status = node.isOnline() ? "online" : "offline";
      console.log(`Node status: ${status}, id: ${this.id.id}`);
      this.me = this.id.id;
      console.log(this.me);
      console.log(node.swarm.peers());

      await node.pubsub.subscribe("announce-circuit", this.processAnnounce);
      console.log("subscribed to announce-circuit");

      for (let i = 0; i < 10; i++) {
        await node.pubsub.publish(
          "announce-circuit",
          new TextEncoder().encode("peer-alive")
        );
        setTimeout(function(){}, 1000)
      }

      setInterval(function () {
        if (this.peerCount < 100) {
          node.pubsub.publish(
            "announce-circuit",
            new TextEncoder().encode("peer-alive")
          );
        }
      }, 1000);

      setInterval(function () {
        node.pubsub.publish(
          "announce-circuit",
          new TextEncoder().encode("keep-alive")
        );
      }, 2000);

      

      return node;
    },
    async publishMessages(message) {
      //publish messages to the channel
      this.messages.push(message);
      this.node.pubsub.publish(this.channel, new TextEncoder().encode(message));
    },
    async processAnnounce(announce) {
      // process the recieved address
      const addr = new TextDecoder().decode(announce.data);
      

      if (announce.from == this.me.string) {
        
        return;
      }

      console.log(
        "Announce reeceived " + addr + "from " + announce.from.toString()
      );

      if (addr == "peer-alive") {
        console.log(addr);
        this.peerCount += 1;

        setTimeout(function () {
          this.peerCount -= 1;
        }, 15000);

        this.lastPeer = new Date().getTime();
      }
      // keep-alives are also sent over here, so let's update that global first
      this.lastAlive = new Date().getTime();

      if (addr == "keep-alive") {
        console.log(addr);
        this.lastAlive = new Date().getTime();
        return;
      }

      // get a list of peers
      this.peers = await this.node.swarm.peers();
      console.log(this.peers);
      for (let i in this.peers) {
        // if we're already connected to the peer, don't bother doing a circuit connection
        if (this.peers[i].peer == this.peer) {
          return;
        }
      }
      // log the address to console as we're about to attempt a connection
      console.log(addr);
      // connection almost always fails the first time, but almost always succeeds the second time, so we do this:
      try {
        await this.node.swarm.connect(announce.from);
      } catch (err) {
        console.log(err);
        await this.node.swarm.connect(announce.from);
      }
    },
    async checkAlive() {
      let now = new Date().getTime();
      if (now - this.lastAlive >= 35000) {
        //TODO restart the  node if it is not connected to the network
        if (now - this.lastPeer >= 35000) {
          console.log("No peers found, restarting node");
          await this.node.stop();
          this.node = await this.createNode();
        } else {
          console.log("No peers found, restarting node");
          await this.node.stop();
          this.node = await this.createNode();
          console.log(this.node);
          this.setChannel("global");
          setInterval(this.checkAlive, 60000);
        }
        // sometimes we appear to be connected to the bootstrap nodes, but we're not, so let's try to reconnect
      } else {
        console.log("Still connected to bootstrap nodes");
      }
    },
  },

  async created() {
    const node = await this.createNode();
    console.log(node);
    this.setChannel("global");
    setInterval(this.checkAlive, 60000);
    
  },
  mounted() {},
}
</script>
