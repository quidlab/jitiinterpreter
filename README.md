# jitiinterpreter
Add Multi Lingual capability to jitsi
This works and has been tested with jwt implementation. In my case i use iframe API and users login to a separate url ( other than jitsi server). Users are issued jwt token. On this page add a toggle button with following code.
````
<div class="right1" id="foqus"></div>
<div id="draggable" class="m-2 top"><input id="selectlanguage" type="checkbox" checked data-toggle="toggle" data-on="Live" data-off="Interpretation" data-onstyle="success" data-offstyle="danger"></div>
</div>

````

add following scripts on same page, interpreter ( 2nd language) login and name should start with interpret ( or must contain this word)
````
const interpreter_pattern = /interpret/i;
/////////////// Interpretation///////


		api.addListener('participantJoined', participantJoined);

		function getInterpreterID() {
			var participants = api.getParticipantsInfo();
			console.log(this.api.getParticipantsInfo());
			var interpreterID;
			// var participantsList = [];
			participants.forEach(function(value, index, arr) {
				//  participantsList.push(value.displayName);
				console.log("getting interpretter: " + value.displayName);
				console.log("getting interpretter: " + value.participantId);
				if (isInterpreter(value.displayName)) {
					//console.log("Problem 2 not here" + value.participantId );
					interpreterID = value.participantId;
					//return interpreterID;
				}

			});
			return interpreterID;
		}





		function participantJoined(arg) {
			console.log("Someone joined: ", arg.displayName);
			InterpreterJoined = isInterpreter(arg.displayName);
			console.log('inter joined ' + InterpreterJoined);
			// if new participant is interpreter and button is on Live then mute audio interpreter
			if (isLive && InterpreterJoined) {
				var interpreterID = arg.participantId;
				var data = {
					'isFoQus': true,
					'isLive': true,
					'InterpreterID': interpreterID
				};
				postMsg(data);
			}
			if (!isLive) {
				var interpreterID = getInterpreterID();
				var data = {
					'isFoQus': true,
					'isLive': false,
					'InterpreterID': interpreterID
				};
				postMsg(data);
			}

		}



		function isInterpreter(displayName) {
			console.log("Testing if interpreter: " + displayName);
			if (interpreter_pattern.test(displayName)) {
				console.log("Is interpreter");
				return true;
			} else {
				console.log("Is not interpreter");
				return false;
			}
		}

function isLive() {
			selectedlanguage = document.getElementById('selectlanguage').checked;
			if (selectedlanguage) {
				return true;
			} else {
				return false;
			}
		}
		const jitsi_server = "https://your.jitsi.server.com" // your jutsi server

		function postMsg(data) {

			var iframeWin = document.getElementById("jitsiConferenceFrame0").contentWindow;
			iframeWin.postMessage(data, "https://your.jitsi.server.com");
		}
	</script>
	<script>

		function CheckEveryOneSec() {

			selectedlanguage = document.getElementById('selectlanguage').checked;
			if (selectedlanguage) {
				var data;
				var InterpreterID = getInterpreterID();
				data = {
					'isFoQus': true,
					'isLive': true,
					'InterpreterID': InterpreterID
				};
				postMsg(data);
			} else {
				var data;
				var InterpreterID;
				InterpreterID = getInterpreterID();
				data = {
					'isFoQus': true,
					'isLive': false,
					'InterpreterID': InterpreterID
				};
				postMsg(data);

			}
		}
		var myInterval = setInterval(CheckEveryOneSec, 2000);
	</script>

		//////////////////////////////////////////

````
on jitsi-meet index page add folloiwng;
````
<script src="interpret.js"></script>
````

interpret.js file shown below

''''''''''''''''''
const interpreter_pattern =  /interpret/i	;
//data={'isFoQus':true,'isLive':false,'InterpreterID':InterpreterID};
function receivedMessage(evt) {
    console.log("message received");
	console.log(evt.data);
    if (!evt.data.hasOwnProperty('isFoQus')) {
        console.log('Not FoQus');
        return false;
    }
	var referrerDomain=evt.origin;
    if (referrerDomain.substr(referrerDomain.length - 8) !== "foqus.vc") {
        console.log(referrerDomain);
        return false;
    } else {


        if (evt.data.isLive == true) {
            console.log("Live Triggred");
            console.log(evt.data.InterpreterID);
			//elem='remoteAudio_'+participantId+'-';
			//setTimeout(unMuteAll, 2000);
			unMuteAll();
			muteInterPreter(evt.data.InterpreterID);
			


        } else {
            console.log("InterPreter Triggered");

			muteAll();
			unMuteInterPreter(evt.data.InterpreterID);

/*             participants = APP.conference.listMembers();
            participants.forEach(function (value, index, arr) {
                console.log('Name:' + value._displayName);
                console.log('ID:' + value._id);

            }) */

        }
    }

}

function unMuteAll(){
	getaudios = document.querySelectorAll('[id^="remoteAudio_"]');
	console.log(getaudios);
	getaudios.forEach((remoteAudio) => {
	remoteAudio.muted=false;
	remoteAudio.play();
	});
}

function muteAll() {
	//elem='remoteAudio_';
	getaudios = document.querySelectorAll('[id^="remoteAudio_"]');
	console.log(getaudios);
	getaudios.forEach((remoteAudio) => {
	remoteAudio.muted=true;
	remoteAudio.pause();
});
	
}

function unMuteInterPreter(id) {
	//elem='remoteAudio_'+id+'-audio-2';
	elem='remoteAudio_'+id;
	//remoteAudio_75f0c7f5-audio-2
	console.log('UnmuteInterPreter ' +elem);
	getaudio = document.querySelector('[id^='+elem+']');
	//getaudio = document.querySelector(elem);
	if (!getaudio) { console.log('not found ' + getaudio); return false;}
	console.log(getaudio);
	getaudio.muted=false;
	getaudio.play();
	
}

function muteInterPreter(id) {
	elem='remoteAudio_'+id;
	console.log('muteInterPreter ' +elem);
	getaudio = document.querySelector('[id^='+elem+']');
	if (!getaudio) { return false;}
	getaudio.muted=true;
	getaudio.pause();
	
}
window.addEventListener("message", receivedMessage, false);
/* for recording */
function setRecorderSettings() {
	try {
		var isRecorder = APP.store.getState()["features/base/config"].iAmRecorder;
	} catch(e) 
	{
		console.log('Can not Accesss getState');
		setTimeout(function() {setRecorderSettings();}, 1000);
	}

	console.log('isRecorder Value :'+isRecorder);
   if (!isRecorder ) return false;
	try {

	const plist= APP.conference.listMembers();
	console.log(plist);
	plist.forEach(function(value) {
		console.log('Each participant value:');
		console.log(value._displayName);
		console.log(value._id);
		if (interpreter_pattern.test(value._displayName)) { muteInterPreter(value._id) }
	});
	
	} catch(e) {
    // do nothing
	  } finally {
    setTimeout(function() {setRecorderSettings();}, 3000);
  }

}

//setTimeout(function() {setRecorderSettings();}, 1000);
setRecorderSettings();

''''''''''''''''''

