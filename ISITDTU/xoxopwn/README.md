# xoxopwn
nc 178.128.12.234 10002  
or  
nc 178.128.12.234 10003

After connecting we see this:

	This is function x()>>>

We're obviously in an interpreter, let's see if we can figure out the language:

	This is function x()>>> __file__
	/home/xoxopwn/xoxopwn.py

Looks like this is python, `__file__` returns the name of the source file, let's try to read it:

	This is function x()>>> open(__file__).read()
	import socket
	import threading
	import SocketServer

	host, port = '0.0.0.0', 9999

	def o(a):
		secret = "392a3d3c2b3a22125d58595733031c0c070a043a071a37081d300b1d1f0b09"
		secret = secret.decode("hex")
		key = "pythonwillhelpyouopenthedoor"
		ret = ""
		for i in xrange(len(a)):
			ret += chr(ord(a[i])^ord(key[i%len(a)]))
		if ret == secret:
			print "Open the door"
		else:
			print "Close the door"


	def x(a):
		xxx = "finding secret in o()"
		if len(a)>21:
			return "Big size ~"
		#print "[*] ",a
		return eval(a)

	class ThreadedTCPServer(SocketServer.ThreadingMixIn, SocketServer.TCPServer):
	    allow_reuse_address = True

	class ThreadedTCPRequestHandler(SocketServer.BaseRequestHandler):
	    def handle(self):
	    	self.request.sendall("This is function x()")
	        self.request.sendall(">>> ")
	        self.data = self.request.recv(1024).strip()
	        print "{} wrote: {}".format(self.client_address[0],self.data)
	        ret = x(str(self.data))
	        self.request.sendall(str(ret))

	if __name__ == "__main__":
		serverthuong123 = ThreadedTCPServer((host, port), ThreadedTCPRequestHandler)
		server_thread = threading.Thread(target=serverthuong123.serve_forever)
		server_thread.daemon = True
		server_thread.start()
		print "Server loop running in thread:", server_thread.name
		server_thread.join()

Looking at the `x` function, we can see that our input is sent directly to `eval` after making sure it's no longer than 21 characters.  
But the `o` function seems a little more interesting, it's parameter is xor'd with the string "pythonwillhelpyouopenthedoor" and then tested to equal "392a3d3c2b3a22125d58595733031c0c070a043a071a37081d300b1d1f0b09" (in hex).  
So we need to find a string `s` where this check succeeds, this is simply done by xoring the two strings we already have:

	secret = "392a3d3c2b3a22125d58595733031c0c070a043a071a37081d300b1d1f0b09".decode("hex")
	key = "pythonwillhelpyouopenthedoor"
	result = ""
	for i in xrange(len(secret)):
	        result += chr(ord(secret[i]) ^ ord(key[i % len(key)]))
	print result

flag: `ISITDTU{1412_secret_in_my_door}`
