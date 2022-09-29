import socket
import sys
import re

# Send and receive protocol"
def send(sock, msg):
  msg = msg + "\r\n"
  sock.send(msg.encode("utf-8"))
  receive = sock.recv(1024).decode("utf-8")
  return receive

# Set up the the socket connection 
def login(ftp):
  username = re.search(r'ftp://(.*?):', ftp).group(1)
  password = re.search(r':(.*?)@', ftp).group(1)
  password = re.search('(?<=:).*', password).group(0)
  host = re.search(r'@(.*?):', ftp).group(1)
  port = ftp.split(":", 3)[-1]
  port = port.split("/")[0]
  
  cSock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
  ip = socket.gethostbyname(host)
  cSock.connect((ip, int(port)))
  msg = cSock.recv(2048)
  print(msg)

  # Set the server up
  msg = send(cSock, "USER " + username)
  print(msg)
  msg = send(cSock, "PASS " + password)
  print(msg)
  msg = send(cSock, "TYPE I")
  print(msg)
  msg = send(cSock, "MODE S")
  print(msg)
  msg = send(cSock, "STRU F")
  print(msg)

  return cSock

# Open a data channel
def openData(cSock, host):
  msg = send(cSock, "PASV")
  print(msg)
  start = msg.find("(")
  end = msg.find(")")
  pTuple = msg[start+1:end].split(',')
  port = int(pTuple[4])*256 + int(pTuple[5])

  ip = socket.gethostbyname(host)
  dSock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
  dSock.connect((ip, port))
  return dSock

# Run a command
def protocol(cSock, host, fullCommand):
  dSock = openData(cSock, host)
  msg = send(cSock, fullCommand)
  print(msg)
  msg = dSock.recv(2048)
  print(msg)
  dSock.close()
  msg = cSock.recv(2048)
  print(msg)
  
###############################################################################
if len(sys.argv) == 3:
  cmd = sys.argv[1]
  ftp = sys.argv[2]
  host = re.search(r'@(.*?):', ftp).group(1)
  path = ftp.split("/", 3)[-1]

  # open client socket
  cSock = login(ftp)

  # command handling
  if cmd == "ls":
    protocol(cSock, host, "LIST " + path)
  elif cmd == "rm":
    msg = send(cSock, "DELE " + path)
    print(msg)
  elif cmd == "mkdir":
    msg = send(cSock, "MKD " + path)
    print(msg)
  elif cmd == "rmdir":
    msg = send(cSock, "RMD " + path)
    print(msg)
  else: 
    print("Error")
  
  # closing sockets
  send(cSock, "QUIT")
  cSock.close()
  
elif len(sys.argv) == 4:
  cmd = sys.argv[1]
  # ftp to local or local to ftp
  if "ftp" in sys.argv[2]:
    ftp = sys.argv[2]
    local = sys.argv[3]
    ftl = True
  else:
    ftp = sys.argv[3]
    local = sys.argv[2]
    ftl = False
  host = re.search(r'@(.*?):', ftp).group(1)
  path = ftp.split("/", 3)[-1]
  
  # open client socket
  cSock = login(ftp)
  
  # command handling
  if ftl:
    # copy from ftp to local
    dSock = openData(cSock, host)
    msg = send(cSock, "RETR " + path)
    print(msg)
    data = dSock.recv(2048)
    print(data)

    # write to local
    file = open(local, "wb")
    file.write(data)
    file.close()
    
    dSock.close()
    msg = cSock.recv(2048)
    print(msg)

    # if mv then delete the original
    if cmd == "mv":
      msg = send(cSock, "DELE " + path)
      print(msg)
  else:
    # copy file from local to ftp 
    file = open(local)
    data = file.read()
    dSock = openData(cSock, host)
    msg = send(cSock, "STOR " + path)
    print(msg)
    dSock.send(data.encode("utf-8"))
    dSock.close()
    msg = cSock.recv(2048)
    print(msg)
    
    # if mv then delete the original
    if cmd == "mv":
      msg = send(cSock, "DELE " + path)
      print(msg)
      
  # Closing sockets
  send(cSock, "QUIT")
  cSock.close()
else:
  print("Bye")
