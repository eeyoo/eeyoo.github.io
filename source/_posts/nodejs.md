title: Node TCP服务器
date: 2015-11-09 21:51:15
categories: Nodejs
tags: Nodejs
---
### TCP聊天服务器源码
``` JavaScript
// chat.js
var net = require('net')

var chatServer = net.createServer(),
	clientList = []

chatServer.on('connection', function(client) {
	// client name
	client.name = client.remoteAddress + ':' + client.remotePort
	client.write('Hi ' + client.name + '!\n')
	console.log(client.name + ' joined!')
	
	clientList.push(client)
	
	// broadcast client message
	client.on('data', function(data) {
		broadcast(data, client);
	})
	
	// splice quit client and print log
	client.on('end', function() {
		console.log(client.name + ' quit');
		clientList.splice(clientList.indexOf(client), 1)
	})
	
	// error log
	client.on('error', function(e) {
		console.log(e)
	})
})

// client broadcast logic
function broadcast(message, client) {
	var cleanup = []
	for(var i=0; i<clientList.length; i++) {
			// broadcast message to others
			if(client !== clientList[i]) {
				// check socket is writable or not
				if(clientList[i].writable) {
					clientList[i].write(client.name + " says " + message)
				} else {
					cleanup.push(clientList[i])
					clientList[i].destroy()
				}
			}
		}
}

chatServer.listen(9000)
console.log('Server started...')
```

### Telnet输出
client 1
``` bash
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
Hi ::ffff:127.0.0.1:54465!
```
client 2
``` bash
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
Hi ::ffff:127.0.0.1:55086!
```

### TCP Server log
``` bash
Server started...
::ffff:127.0.0.1:54465 joined!
::ffff:127.0.0.1:55086 joined!
```


### 说明
 - 基于TCP的聊天服务器
 - Telnet客户端访问服务器
 - 需要理解客户端断开导致服务器崩溃的原理