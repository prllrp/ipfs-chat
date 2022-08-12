<template>
  <div class="main">
    <Chat :messages ="messages" @send-message = "sendMessage" @join-channel = "setChannel"/>
  </div>
</template>

<script>
import Chat from '~/components/Chat.vue';
export default {
    name: "IndexPage",
    components: { Chat },
    data(){
    return {
      node: null,
      messages: [],
      message: '',
      peers: [],
      peer: null,
      peerCount:  0,
      channel: null,
      lastAlive: 0,
      lastPeer: 0,
      bootstraps : [],
      lastBootstrap: 0,
      id: null,
      channelJoined: false,
      chatPeers:  [],

    }
  },
  methods: {
    async setChannel(channel){
      this.channel = channel;
      console.log('Channel set to ' + channel);
      await this.node.pubsub.subscribe(this.channel, (data) => {
        let message = JSON.parse(new TextDecoder().decode(data.data));
        this.messages.push(message);
        console.log(this.messages);
        this.title = channel;
      }
      );
      this.channelJoined = true;
    },
    async sendMessage(message) {
      await this.node.pubsub.publish(this.channel, new TextEncoder().encode(JSON.stringify({text: message, time: new Date().toLocaleTimeString(), from: this.id,  messageId: Math.floor(Math.random()*1000000)})));
    },
    async createNode() {//create a node to be passed into the chat component
      const node = await Ipfs.create({
        repo: 'ipfs-' + Math.random(),
        relay: {
				enabled: true,
				hop: {
					enabled: true
				}
			},
        config: {
          Addresses: {
            Swarm: [
              '/dns4/wrtc-star1.par.dwebops.pub/tcp/443/wss/p2p-webrtc-star',
              '/dns4/wrtc-star2.sjc.dwebops.pub/tcp/443/wss/p2p-webrtc-star',
              '/dns4/star.thedisco.zone/tcp/9090/wss/p2p-webrtc-star',
              '/dns6/star.thedisco.zone/tcp/9090/wss/p2p-webrtc-star',
              '/dns6/ipfs.thedisco.zone/tcp/4430/wss/p2p/12D3KooWChhhfGdB9GJy1GbhghAAKCUR99oCymMEVS4eUcEy67nt', 
              '/dns4/ipfs.thedisco.zone/tcp/4430/wss/p2p/12D3KooWChhhfGdB9GJy1GbhghAAKCUR99oCymMEVS4eUcEy67nt'
            ]
          }
        }
      });
      console.log(node);
      this.id = await node.id().id
      this.node = node;
      const status = node.isOnline() ? 'online' : 'offline'
      const id = await node.id()
      this.id = id.id
      console.log(`Node status: ${status}, id: ${id.id}`)
      this.me = id.id
      console.log(this.me)
      console.log(node.swarm.peers())

      await node.pubsub.subscribe('announce-circuit', this.processAnnounce)
      console.log('subscribed to announce-circuit')


      for(let i = 0; i < 10; i++){
        await node.pubsub.publish('announce-circuit', new TextEncoder().encode('peer-alive'))
      }
      

      setInterval(function () {
        if(this.peerCount < 100){
          node.pubsub.publish('announce-circuit', new TextEncoder().encode('peer-alive'))
        }
      }, 1000);

      setInterval(function () {
        node.pubsub.publish('announce-circuit', new TextEncoder().encode('keep-alive'))
      }, 2000);

      
      
      return node;
    },
    async publishMessages(message) {
      this.messages.push(message);
      this.node.pubsub.publish(this.channel, new TextEncoder().encode(message))
    },
    async processAnnounce(announce) {
			
			
			// process the recieved address
			const addr = new TextDecoder().decode(announce.data);
			console.log('Announce reeceived ' + addr +  'from ' + announce.from.toString());
      
      
      if(announce.from == this.me.string){
        console.log('Ignoring self announcement')
        //return;
      }
			
			if (addr == "peer-alive") {
				console.log(addr);
				this.peerCount += 1;
				
				setTimeout(function(){
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
			} catch(err) {
				console.log(err);
				await this.node.swarm.connect(announce.from);
			}
			
		},
    async checkAlive() {
			let now = new Date().getTime();
			if (now-this.lastAlive >= 35000) {
        //TODO restart the  node if it is not connected to the network
				if (now-this.lastPeer >= 35000) {
					console.log('No peers found, restarting node');
          await this.node.stop();
          this.node = await this.createNode();
				} else {
					console.log('No peers found, restarting node');
          await this.node.stop();
          this.node = await this.createNode();
				}
				// sometimes we appear to be connected to the bootstrap nodes, but we're not, so let's try to reconnect
			} else {
				console.log('Still connected to bootstrap nodes');
			}
		},
  },

  async created() {
    const node = await this.createNode();
    console.log(node);
    this.setChannel('global');
    setInterval(this.checkAlive, 60000);
  },
  mounted() {
    }
  }

</script>
