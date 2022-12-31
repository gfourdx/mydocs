# TCP通讯

## TCP_Server

```python
import socket

tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_server.bind(('localhost', 8080))
tcp_server.listen(5)
tcp_client, tcp_client_ip_port = tcp_server.accept()
tcp_client_data = tcp_client.recv(1024).decode('utf-8')
host = tcp_client_ip_port[0] + ':' + str(tcp_client_ip_port[1])
print(host+':',tcp_client_data)
tcp_client.send('服务端已收到客户端发送的消息！'.encode('utf-8'))

```

## TCP_Client

```python
import socket

tcp_client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_client.connect(('localhost', 8080))
tcp_client.send('我是tcp客户端！'.encode('utf-8'))
print(tcp_client.recv(1024).decode('utf-8'))

```

## TCP_Server_Client

```python
import socket


def send_data(tcp_socket, tcp_address):
    tcp_socket.connect(tcp_address)
    tcp_socket.sendto(input('请输入要发送的内容：').encode('utf-8'), tcp_address)
    print(tcp_socket.recv(1024).decode('utf-8'))


def recv_data(tcp_socket, tcp_address):
    tcp_socket.bind(tcp_address)
    tcp_socket.listen(5)
    tcp_client, tcp_client_ip_port = tcp_socket.accept()
    tcp_client_data = tcp_client.recv(1024).decode('utf-8')
    host = tcp_client_ip_port[0] + ':' + str(tcp_client_ip_port[1])
    print(host + ':', tcp_client_data)
    tcp_client.send('服务端已收到客户端发送的消息！'.encode('utf-8'))


if __name__ == '__main__':
    tcp_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    tcp_address = ('localhost', 8080)
    # 不经历TIME_WAIT的过程, 立即关闭socket并释放端口
    tcp_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
    while True:
        choice = input('1、发送\t2、接收\n')
        if choice == '1':
            send_data(tcp_socket, tcp_address)
        if choice == '2':
            recv_data(tcp_socket, tcp_address)

```

