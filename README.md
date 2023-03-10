<!DOCTYPE html>
<html>
<head>
	<title>Chat Room</title>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<link rel="stylesheet" href="style.css">
</head>
<body>
	<h1>Chat Room</h1>
	<div id="chatbox"></div>
	<input type="text" id="message">
	<button onclick="sendMessage()">Send</button>
	<input type="file" id="fileInput">
	<button onclick="uploadFile()">Upload</button>
	<video id="video"></video>
	<button onclick="startVideo()">Start Video</button>
	<button onclick="stopVideo()">Stop Video</button>
	<script>
        var signalingChannel = new WebSocket("ws://localhost:12345");

var peerConnection = new RTCPeerConnection();

navigator.mediaDevices.getUserMedia({video: true, audio: true}).then(function(stream) {
  var localVideo = document.createElement("video");
  localVideo.srcObject = stream;
  localVideo.play();
  peerConnection.addStream(stream);
});

peerConnection.onaddstream = function(event) {
  var remoteVideo = document.getElementById("video");
  remoteVideo.srcObject = event.stream;
  remoteVideo.play();
};

signalingChannel.onmessage = function(event) {
  var message = JSON.parse(event.data);
  if (message.type === "offer") {
    peerConnection.setRemoteDescription(new RTCSessionDescription(message));
    peerConnection.createAnswer().then(function(description) {
      peerConnection.setLocalDescription(description);
      signalingChannel.send(JSON.stringify(description));
    });
  } else if (message.type === "answer") {
    peerConnection.setRemoteDescription(new RTCSessionDescription(message));
  } else if (message.type === "icecandidate") {
    peerConnection.addIceCandidate(new RTCIceCandidate(message.candidate));
  } else if (message.type === "message") {
    var chatbox = document.getElementById("chatbox");
    chatbox.innerHTML += "<p>" + message.content + "</p>";
  } else if (message.type === "file") {
    // handle file received from other user
  }
};

function sendMessage() {
  var message = document.getElementById("message").value;
  var data = {type: "message", content: message};
  signalingChannel.send(JSON.stringify(data));
  var chatbox = document.getElementById("chatbox");
  chatbox.innerHTML += "<p>" + message + "</p>";
}

function uploadFile() {
  var fileInput = document.getElementById("fileInput");
  var file = fileInput.files[0];
  var reader = new FileReader();
  reader.onload = function(e) {
    var fileContent = e.target.result;
    var data = {type: "file", content: fileContent};
    signalingChannel.send(JSON.stringify(data));
  }
  reader.readAsDataURL(file);
}

function startVideo() {
  peerConnection.createOffer().then(function(description) {
    peerConnection.setLocalDescription(description);
    signalingChannel.send(JSON.stringify(description));
  });
}

function stopVideo() {
  var remoteVideo = document.getElementById("video");
  remoteVideo.pause();
  remoteVideo.srcObject = null;
}

    </script>
</body>
</html>
