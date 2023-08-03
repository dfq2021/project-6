# project-6
impl this protocol with actual network communication
# 问题重述
![9699f01708bbe54e12fad6b31803570](https://github.com/jlwdfq/project-6/assets/129512207/c8c8f1d4-1f72-45b9-bc49-997a46931506)
根据上图描述，通过真实网络传播实现该协议。
# 实现思路
该例子主要是应用到了hash-chain和签名。hash-chain即对某一对象多次hash，在本实验实现中，因hash函数接受的是字节流，所以对hash后的结果进行.digest()即可得到字节流。本实验的两个hash函数分别为SHA1和SHA256，签名为RSA签名。为了模拟真实网络传播，我们分成Issuer、Alice、Bob三个角色实现各自的操作。
### Issuer
Issuer作为第三方，生成Issuer的公私钥对并将signature和s发给Alice。

关键代码：
```python
#生成Issuer的公私钥对
random_generator = Random.new().read
rsa = RSA.generate(1024, random_generator)
private_pem = rsa.exportKey()
public_pem = rsa.publickey().exportKey()
with open('private_key.pem', 'wb') as f:
    f.write(private_pem)
with open('public_key.pem', 'wb') as f:
    f.write(public_pem)
#Issuer
seed=random.randint(pow(2,127),pow(2,128))
s=hash_function1(str(seed).encode('utf-8'))
k=2100-1978
current_hash=hash_function2(s.digest())
hash_chain1 = [current_hash]

for i in range(1, k):
    current_hash = hash_function2(current_hash.digest())
    hash_chain1.append(current_hash)
c=hash_chain1[-1]

with open('private_key.pem') as f:
    key=f.read()
    signkey=RSA.importKey(key)
    signer=Signature_pkcs1_v1_5.new(signkey)
    sign=signer.sign(c)
    signature=base64.b64encode(sign)
    # Issuer give Alice s and sig
    s0 = socket.socket()  # 创建 socket 对象
    host = socket.gethostname()  # 获取本地主机名
    port = 12345  # 设置端口
    s0.bind((host, port))  # 绑定端口
    print("等待接收方链接...")
    s0.listen(5)
    c, addr = s0.accept()
    print("检测到成功链接")
    print("-----------", "Issuer give Alice s and sig", "-----------")
    print('s=', s.digest())
    print('signatrure=', signature)
    c.send(s.digest())
    c.send(signature)
s0.close()


```
### Alice
Alice接收到Issuer发送的signature和s后，再将p和signature发给Bob。

关键代码：
```python
#Alice receive s and sig form issuer
so = socket.socket()  # 创建 socket 对象
host = socket.gethostname()  # 获取本地主机名
port = 12345  # 设置端口号
so.connect((host, port))
print("成功链接发送方")
s = so.recv(1024)
signature = so.recv(1024)
signature=signature
print("收到s和sig，分别为：",s,signature)
d0=2000-1978
current_hash=hash_function2(s)
hash_chain2 = [current_hash]
for i in range(1, d0):
    current_hash = hash_function2(current_hash.digest())
    hash_chain2.append(current_hash)
p=hash_chain2[-1]
#Alice give Bob p and sig
s1 = socket.socket()  # 创建 socket 对象
host = socket.gethostname()  # 获取本地主机名
port = 12345  # 设置端口
s1.bind((host, port))  # 绑定端口
print("等待接收方链接...")
s1.listen(5)
c, addr = s1.accept()
print("检测到成功链接")
print("-----------", "Alice give Bob p and sig", "-----------")
print('p=', p.digest())
print('signatrure=', signature)
c.send(p.digest())
c.send(signature)

```
### Bob
Bob接收Alice发来的p和signature，再进行验证
关键代码：
```python

s1 = socket.socket()  # 创建 socket 对象
host = socket.gethostname()  # 获取本地主机名
port = 12345  # 设置端口号
s1.connect((host, port))
print("成功链接发送方")
p = s1.recv(1024)
signature = s1.recv(1024)
signature=signature
print("收到p和sig，分别为：",p,signature)

#Bob
d1=2100-2000
current_hash=hash_function2(p)
hash_chain3 = [current_hash]
for i in range(1, d1):
    current_hash = hash_function2(current_hash.digest())
    hash_chain3.append(current_hash)
c1=hash_chain3[-1]
with open('public_key.pem') as f:
    key=f.read()
    unsignkey=RSA.importKey(key)
    unsigner=Signature_pkcs1_v1_5.new(unsignkey)
    is_verify = unsigner.verify(c1, base64.b64decode(signature))
    print("-----------","Bob verify Alice's proof:",is_verify,"-----------")

```
# 实验结果
### Issuer
![K1B5$K%D}{U)3U}0_M G`J3](https://github.com/jlwdfq/project-6/assets/129512207/44c6d075-29bc-4740-820a-f97456d9f1e1)

### Alice
![HH21TK@2(9C)N69BZZHJV(K](https://github.com/jlwdfq/project-6/assets/129512207/ff885f24-8f47-4150-a755-02d664be91c2)

### Bob
![%N@VFSE`)DHAX@4_@H31TZF](https://github.com/jlwdfq/project-6/assets/129512207/8a052867-62dd-4485-9d6f-107a92831597)
# 实验环境
| 语言  | 系统      | 平台   | 处理器                     |
|-------|-----------|--------|----------------------------|
| Cpp   | Windows10 | pycharm| Intel(R) Core(TM)i7-11800H |
# 小组分工
戴方奇 202100460092 单人组完成project6
