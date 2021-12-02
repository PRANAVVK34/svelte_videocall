<script>
	
	import AgoraRTC from 'agora-rtc-sdk'



	const options = {
		appid : '81d99d5bc7ad4117a6023f4128abdb1f',
		token : null,
		channel : 'video-chat',
		
	}

	let remoteVideos = []



	const joinChannel = async() =>{

		let client = AgoraRTC.createClient({ mode: "live", codec: "vp8" });

		client.init(options.appid, () => {

		client.join(options.token, options.channel, null, (uid) => {
			console.log("join success", uid);

			const localStream = AgoraRTC.createStream({ audio: false, video: true });

			localStream.init(() => {
			console.log("init stream success");

			//localstream
			let localVideo = document.getElementById('video');
			localVideo.srcObject = localStream.stream;

			localVideo.onloadeddata = () =>{
				localVideo.play()
			}

			// localStream.play('video', { muted: true });

				//publish stream
		client.publish(localStream, err => {
				console.log("publish failed", err);
				console.log(localStream);
				});
				client.on("stream-published",  () => {
				console.log("publish success");
				});




			}, (e) => {
			console.log("init local stream failed", e);
			});

			

	
					
		}, (e) => {
			console.log("join failed", e);
		});
		});

		//client connect
		client.on("stream-added", e => {
		client.subscribe(e.stream, { audio: false, video: true }, err => {
			console.log("subscribe failed", err);
		});
		});

		client.on("stream-subscribed", e => {
		console.log("subscribe success",e);

		// e.stream.play('streamvideo');
		let remoteVideoDiv = document.getElementById('streamvideo');

		let streamId = e.stream.getId()

		let streamVideoDom = document.createElement('video')

		remoteVideoDiv.appendChild(streamVideoDom)
		
		streamVideoDom.id=streamId
		streamVideoDom.srcObject=e.stream.stream
		streamVideoDom.style.width="100%"
		streamVideoDom.style.height="100%"
		streamVideoDom.style.objectFit="cover"
		streamVideoDom.style.borderRadius="50%"
		streamVideoDom.muted=true
		streamVideoDom.playsInline = true
		streamVideoDom.autoplay = true

		let newRemoteVideos = [...remoteVideos]
		
	
		newRemoteVideos[streamId] = streamVideoDom
	
		remoteVideos= remoteVideos.push(newRemoteVideos) 
		console.log(remoteVideos);
		let videoDom = document.getElementById(streamId)


		videoDom.srcObject = e.stream.stream

		// remoteVideoDiv.appendChild(videoDom)

		// remoteVideo.srcObject = e.stream.stream;

		videoDom.onloadeddata = () =>{
			videoDom.play()
			}
		
		});
		
	
		
		client.on('stream-removed',e=>{
			console.log("stream-removed");
			leaveChannel(e)

		})     

	

		client.on("peer-leave", function(evt) {
		var uid = evt.uid;
		var reason = evt.reason;
		console.log("remote user left ", uid, "reason: ", reason);
		client.leave(function() {
			console.log("client leaves channel");
			//……
		}, function(err) {
			console.log("client leave failed ", err);
			//error handling
		});
	});

  const leaveChannel = (e) =>{
	 
			let stream = e.stream;
			let streamId = String(stream.getId());
			let remDiv = document.getElementById(streamId)
			console.log(remDiv);
			
			stream.close()
			// remoteVideos = remoteVideos.pop(remoteVideos[streamId])
			// removeVideoStream(streamId)
			
			
			
  }



}
const leave =()=>{
	window.location.reload()
}


</script>

<main>

	<button on:click={joinChannel}>Go Live</button>
	<button on:click={leave}>leave</button>
	
	<div class="streams">
		<!-- <div id="video" style="height: 220px;width: 220px;"></div> -->

		<video id="video" muted autoplay>
			<track kind="captions">
		</video>

		<div id="streamvideo" >
			
		</div>
		
		<!-- <video id="streamvideo" muted autoplay >
			<track kind="captions">
		</video> -->

	</div>

	

</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		margin: 0 auto;
		
	}
	

	#video{
		width: 100px;
		height: 100px;
		border-radius: 50%;
		object-fit: cover;
	}

	#streamvideo{
		width: 100px;
		height: 100px;
	}



	.streams{
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: space-around;
	}





</style>