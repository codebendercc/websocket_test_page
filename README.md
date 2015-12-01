### websocket test.html page

The test.html page can be used to test [windows-driver-installer.exe] (https://github.com/codebendercc/windows-drivers-installer) as well as the [driver-installer.mpkg] (https://github.com/codebendercc/osx-ftdi-driver-installer/tree/codebenderDrivers).

#### How it works
- On Windows

Once the installer is executed we start our node.js server by executing the command:

```ExecCmd::exec  /NOUNLOAD /async  '"$TEMP\codebender\node.exe wsServer.js"'```

Once the server is started a new instance of a websocket server that listens to port 9010 will be created.
By reloading test.html page while the server is up, we will open a connection to that server. Once the connection is opened successfully we log message `CONNECT` to the console through the test.html.

```
ws.onopen = function() {
  log('CONNECT');
};
```

 In the same time, as soon as connection is established, the websocket server sends message `installation started` to the browser.

 When test.html page receives `installation started message` we log message `received message installation started. Waiting...` to the console.

```
if(event.data == "installation started") {
	console.log("received message installation started. Waiting...");
}
```

Back to the websocket server, we are waiting for drivers installation to finish. We search for install.txt, the file that is created once the installer finishes with drivers installation, and if it exists, we search for the word **"success"** or **"failure"**. Then, we call the corresponding function of wsServer.js.

```
if (data.search('success') != -1) {
    console.log(data);
    Success();
    startEventListeners();
    return;
}

if (data.search('failure') != -1) {
    console.log(data);
    Failure(data);
    startEventListeners();
    return;
}
```

For 60 secs the server will send `installation complete` to the browser, if installation was successful, or `installation failure`, if installation was unsuccessful until we receive `installation complete ack` or `installation failed ack` from the browser.

To the browser's side, when `installation complete` or `installation failure` is received, browser responds by sending `installation complete ack` or `installation failed ack` accordingly to the server.

```
if(event.data == "installation complete") {
	ws.send("installation complete ack");
}
if(event.data.indexOf("installation failure") >-1) {
	ws.send("installation failed ack");
}
```

As soon as server receives `installation complete ack` or `installation failed ack` from the browser, responds with `ack ack` and closes the connection.

Once the connection is closed, browser logs message `DISCONNECT` to the console.

```
ws.onclose = function() {
	log('DISCONNECT');
};
```

- On Mac

We start our websocket server that listens to port 9010 by executing the command:

```
"$dir/$BINARY" --ssl --sslcert="$dir/localhost_codebender_cc.crt"  --sslkey="$dir/codebender-localhost.key" --port=9010 "$dir/websock.sh" "$dir/install.txt" "$dir/results.txt" &
```

Then, we follow the same process on Windows; test.html page opens a connection to the server and if the connection is successful message `CONNECT` is logged to the console.
The websocket server sends message `installation started` to the browser and once received, message `received message installation started. Waiting...` appears to the console.
As before, on server's side, we are waiting for drivers installation to finish; we search for the word **"success"** or **"failure"** on install.txt file and we call the corresponding function that sends `installation complete` or `installation failure` to the browser.
To the browser's side, when `installation complete` or `installation failure` is received, browser responds by sending `installation complete ack` or `installation failed ack` accordingly, to the server.
As soon as server receives `installation complete ack` or `installation failed ack` from the browser, responds with `ack ack` and closes the connection.
Once the connection is closed, browser logs message `DISCONNECT` to the console.
