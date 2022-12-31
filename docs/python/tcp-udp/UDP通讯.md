# UDP通讯

## UDP_Server

```python
import socket

# udp服务端
udp_server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
udp_server.bind(('', 8080))

# recv()接收客户端发来的数据
# udp_client_data = udp_server.recv(1024).decode('utf-8')
# print(udp_client_data)

# recvfrom()接收一个元组
# 第一个元送是客户端发来的数据
# 第二个元素是一个包含客户端ip和port的元组
udp_client_data = udp_server.recvfrom(1024)
print(udp_client_data[0].decode('utf-8'), udp_client_data[1])

```

## UDP_Client

```python
import socket

# udp客户端
udp_client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
udp_client.sendto('我是udp客户端'.encode('utf-8'), ('localhost', 8080))

```

## UDP_Server_Client

```python
import socket


def send_data(udp_socket, udp_address):
    udp_socket.sendto(input('请输入要发送的内容：').encode('utf-8'), udp_address)


def recv_data(udp_socket):
    data, ip_port = udp_socket.recvfrom(1024)
    ip, port = ip_port
    host = ip + ':' + str(port)
    print(host + ':', data.decode('utf-8'))


if __name__ == '__main__':
    udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    udp_address = ('localhost', 8081)
    udp_socket.bind(udp_address)
    while True:
        choice = input('1、发送\t2、接收\n')
        if choice == '1':
            send_data(udp_socket, udp_address)
        if choice == '2':
            recv_data(udp_socket)

```

