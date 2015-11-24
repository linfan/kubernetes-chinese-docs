# **认证插件**
Kubernetes使用客户端证书，令牌，或者HTTP基本身份验证用户的API调用。

在API服务器中配置—client-ca-file=SOMEFILE选项，就会启动客户端证书认证。引用文件必须包含一个或多个认证机制，通过认证机制验证传给API服务器的客户端证书。当一个客户端证书通过认证，该证书主题的名字就被作为该请求的用户名。

在API服务器中配置选项：--token-auth-file=SOMEFILE, 启动Token认证。目前，Token没有有效期，必须重启API服务，Token列表的更改才会生效。

令牌文件格式路径：plugin/pkg/auth/authenticator/token/tokenfile/...，该文件是一个CSV文件，含有三行：Token，用户名，用户uid。

当http客户端使用Token认证，apiserver需要含有Bearer Sometoken值的一个Authorization头。

OpenID Connect ID Token，传递下面的参数给apiserver：
- --oidc-issuer-url (必须) API Server连接到OpenID提供者的URL， 只接受HTTPS协议。
- --oidc-client-id (必须) API Server用于验证Token用户，合法的[ID Token](http://openid.net/specs/openid-connect-core-1_0.html#IDToken)在它的aud参数（aud claims 翻译待考虑）中包含该client-id。
- --oidc-ca-file (可选) API Server用于和OpenID提供者建立和验证安全连接。
- --oidc-username-claim (可选, 实验性参数) 指定用户名对应的OpenID。默认设置为sub参数，在指定域中是唯一的，不可变的。集群管理员可以选择其它参数如email，作为用户名，但不保证其唯一性和不变性。

请注意，这个标志仍然处于试验阶段，如果我们可以处理更多关于OpenID用户和Kubernetes用户的映射关系，便可以开始使用。因此，未来的变化还是很有可能的。

目前，该ID Token会通过一些第三方应用程序获取。这意味着应用程序必须和API Server共享该配置--oidc-client-id。

如Token文件，当从HTTP客户端使用Token认证方式，API Server希望在Authorization头添加一个Bearer SOMETOKEN的值。

启动基本认证，需要在API Server配置选项—basic_auth_file=SOMEFILE。当前，基本认证凭据是无限期的，而且重启API Server，密码的修改才会生效。需要注意，基本认证方式是更安全的模式，更容易使用，更通用。

基本认证文件格式，plugin/pkg/auth/authenticator/password/passwordfile/...，该文件是一个CSV文件，含有三个值，密码，用户名和用户id。
如果在HTTP客户端使用基本认证，API Server需要一个值是Basic BASE64ENCODEDUSER:PASSWOR的Authorization头。

Keystone认证会在API Server启动的时候把--experimental-keystone-url='AuthURL'参数传给API Server，该认证就会生效。该插件在plugin/pkg/auth/authenticator/request/keystone/keystone.go文件中实现。有关如何使用Keystone去管理项目和用户的详细信息，请参考[Keystone文档](http://docs.openstack.org/developer/keystone/)。请注意，该插件还处于试验阶段，很可能还会变化。请参考有关该插件的[讨论](https://github.com/kubernetes/kubernetes/pull/11798#issuecomment-129655212)和[计划](https://github.com/kubernetes/kubernetes/issues/11626)了解更多细节。

## **插件开发** 
我们计划给Kubernetes API Server解决Token问题。使用“bedrock”认证用户，外部提供者给Kubernetes。我们计划使Kubernetes和一个Bedrock认证提供者（如github.com，google.com，Enterprise Directory, Kerberos等等）之间的接口开发更容易。

## **附录**
### **创建证书 **
客户端证书认证，用户可以手动产生证书，也可以使用已经存在的脚本部署。

部署脚本路径在cluster/saltbase/salt/generate-cert/make-ca-cert.sh。执行该脚本需要两个参数，一个是API Server的IP地址，另一个是IP：<ip-address>或者DNS：<dns-name>主题备用名称的列表。该脚本会产生三个文件，ca.crt，server.crt和server.key。最后，添加下面的参数作为API Server的启动参数， --client-ca-file=/srv/kubernetes/ca.crt，--tls-cert-file=/srv/kubernetes/server.cert，--tls-private-key-file=/srv/kubernetes/server.key。

easyrsa can be used to manually generate certificates for your cluster.

1.	Download, unpack, and initialize the patched version of easyrsa3.
curl -L -O https://storage.googleapis.com/kubernetes-release/easy-rsa/easy-rsa.tar.gz tar xzf easy-rsa.tar.gz cd easy-rsa-master/easyrsa3 ./easyrsa init-pki
2.	Generate a CA. (--batch set automatic mode. --req-cn default CN to use.)
./easyrsa --batch "--req-cn=${MASTER_IP}@date +%s" build-ca nopass
3.	Generate server certificate and key. (build-server-full [filename]: Generate a keypair and sign locally for a client or server)
./easyrsa --subject-alt-name="IP:${MASTER_IP}" build-server-full kubernetes-master nopass
4.	Copy pki/ca.crt pki/issued/kubernetes-master.crt pki/private/kubernetes-master.key to your directory.
5.	Remember fill the parameters --client-ca-file=/yourdirectory/ca.crt --tls-cert-file=/yourdirectory/server.cert--tls-private-key-file=/yourdirectory/server.key and add these into apiserver start parameters.

openssl can also be use to manually generate certificates for your cluster.
1.	Generate a ca.key with 2048bit openssl genrsa -out ca.key 2048
2.	According to the ca.key generate a ca.crt. (-days set the certificate effective time). openssl req -x509 -new -nodes -key ca.key -subj "/CN=${MASTER_IP}" -days 10000 -out ca.crt
3.	Generate a server.key with 2048bit openssl genrsa -out server.key 2048
4.	According to the server.key generate a server.csr. openssl req -new -key server.key -subj "/CN=${MASTER_IP}" -out server.csr
5.	According to the ca.key, ca.crt and server.csr generate the server.crt. openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 10000
6.	View the certificate. openssl x509 -noout -text -in ./server.crt Finally, do not forget fill the same parameters and add parameters into apiserver start parameters.
