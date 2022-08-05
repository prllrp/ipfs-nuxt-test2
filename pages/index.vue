<template>
  <Tutorial/>
</template>

<script>  

async function createNode () {
  const node = await window.IpfsCore.create({ repo: 'ipfs-' + Math.random() ,
    config: {relay: {
				enabled: true,
				hop: {
					enabled: true
				}
			},
        Addresses: {
          Swarm: [
            '/dns4/wrtc-star1.par.dwebops.pub/tcp/443/wss/p2p-webrtc-star',
            '/dns4/wrtc-star2.sjc.dwebops.pub/tcp/443/wss/p2p-webrtc-star',
            '/dns4/star.thedisco.zone/tcp/9090/wss/p2p-webrtc-star', 
            '/dns6/star.thedisco.zone/tcp/9090/wss/p2p-webrtc-star' 
          ]
        }
      }});
  console.log(node);
  const status = node.isOnline() ? 'online' : 'offline'
  const id = await node.id()
  
  console.log(`Node status: ${status}, id: ${id.id}`)
  console.log(node.swarm.peers())
  return node;
}

async function processAnnounce(announce) {
			
			// process the recieved address
			const addr = new TextDecoder().decode(announce.data);
      console.log(addr)
			
			if (addr == "peer-alive") {
				console.log(addr);
				peerCount += 1;
				
				setTimeout(function(){
					peerCount -= 1;
				}, 15000);
					
				const lastPeer = new Date().getTime();
				return;
			}

			// keep-alives are also sent over here, so let's update that global first
			const lastAlive = new Date().getTime();

			
			if (addr == "keep-alive") {
				console.log(addr);
				return;
			} 
			peer = addr.split("/")[9];
			console.log("Peer: " + peer);
			console.log("Me: " + me);

			// get a list of peers
			peers = await ipfs.swarm.peers();
			for (i in peers) {
				// if we're already connected to the peer, don't bother doing a circuit connection
				if (peers[i].peer == peer) {
					return;
				}
			}
			// log the address to console as we're about to attempt a connection
			console.log(addr);

			// connection almost always fails the first time, but almost always succeeds the second time, so we do this:
			try {
				await ipfs.swarm.connect(addr);
			} catch(err) {
				console.log(err);
				await ipfs.swarm.connect(addr);
			}
		}



async function doStuff(){
  const ipfs = await createNode()
  await ipfs.pubsub.subscribe('pubsub-test', (msg) => {
    console.log(msg)
  })

  setInterval(function(){
    ipfs.pubsub.publish('pubsub-test', new TextEncoder().encode({id: ipfs.id(), time: new Date().getTime(), msg: "peer-alive"}))
  }, 15000);
  
  await ipfs.pubsub.subscribe('announce-circuit', processAnnounce)
  setInterval(function(){
    ipfs.pubsub.publish('announce-circuit', new TextEncoder().encode('keep-alive'))
  }, 1000)
  




  
}
export default {
  name: 'IndexPage',
  components:{
    
  },
  mounted(){
    setTimeout(()=>{console.log(wait)}, 50000)
    doStuff()
  }
}
</script>
