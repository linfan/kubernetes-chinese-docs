# 授权插件
`译者：Nancy` `校对：无`


授权是Kubernetes认证中的一个独立部分，参考[认证部分](http://kubernetes.io/v1.1/docs/admin/authentication.html)的文档。

授权应用在主（安全）API服务端口的所有HTTP访问请求。

任何请求的授权检查都会通过访问策略比较该请求上下文的属性，（比如用户，资源和Namespace）。API的调用必须符合一些规则，按顺序执行。

下面的选项都是可行的，可通过标志位选择：

-	--authorization-mode=AlwaysDeny
-	--authorization-mode=AlwaysAllow
-	--authorization-mode=ABAC

AlwaysDeny会阻止所有的请求（测试中使用的）。AlwaysAllow允许所有请求，如果不需要授权，可以使用这个参数。ABAC（Attribute-Based Access Control）用于已经配置的用户授权规则。

## ABAC模式
### 请求属性
一个授权请求可配置五个参数：

- 用户（用户是否是用户串）
- 组（用户所属组名的列表）
- 请求只读性(GETs是只读的)
- 资源权限
 - 仅适用于API端点，例如/api/v1/namespaces/default/pods。对于杂端点（miscellaneous endpoints，翻译有待考虑），如/version，是空串。
- 可访问对象的Namespace，空串的Namespace，该空串端点不支持命名对象。

我们期望增加更多的属性，允许更细粒度的访问控制，并协助策略管理。

### 策略文件格式
ABAC模式，也可以指定参数：--authorization-policy-file=SOME_FILENAME

该文件格式每行有[一个JSON对象]( http://jsonlines.org/)，不会有封闭列表或者映射，而仅仅每行有一个映射。

每行是一个“策略对象”。一个策略对象是包含以下属性的一个映射：

- user， 字符型；来自--token-auth-file，如果指定用户，必须匹配认证用户的用户名。
- group，字符型；如果指定用户组，必须匹配认证用户所属组的其中一个。
- readonly，布尔型，true表示该策略仅适用于GET操作。
- resource，字符型；一个资源来自一个URL，比如Pod。
- namespace，字符型；一个Namespace字符串。

未设置的属性同类型设置为zero的属性（如，空串，0，false），含义是相同的。但是，没设置的属性首选是可读性。

在未来，策略可以展现在JSON格式中，并通过REST接口进行管理。

### 授权算法
一个请求属性和一个策略对象的属性是相关的。

一个请求被接受时，其属性是确定的。未知属性默认设置为0，如空串，0，false。

未设置的属性将匹配相应属性的任何值。

检查属性的元组在每一个策略文件中每个策略的匹配正确性。至少有一行匹配到该请求属性，则授权该请求（但之后的验证也许会失败）。

为了让每个用户都做些事情，编写一个策略用于用户未设置的属性。（To permit any user to do something, write a policy with the user property unset. To permit an action Policy with an unset namespace applies regardless of namespace. 翻译有待考虑）

###  **示例**
1.	Alice能够做任何事情： {“user”:”alice”}
2.	Kubelet能够读任何pods：{"user":"kubelet", "resource": "pods", "readonly": true}
3.	Kubelet能够读写事件：{“user”: "kubelet", "resource": "events"}
4.	Bob只能在”projectCaribou”命名空间中读pods：{"user":"bob", "resource": "pods", "readonly": true, "ns": "projectCaribou"}

[完整文件示例]( https://github.com/kubernetes/kubernetes/blob/release-1.1/pkg/auth/authorizer/abac/example_policy_file.jsonl)

### 服务账户的快速标记
一个服务账户会自动生成一个用户。该用户的名字生成是根据如下命名规范：
```
system:serviceaccount:<namespace>:<serviceaccountname>
```
创建一个新的Namespace，也会附带创建一个新的服务账户，格式如下：
```
system:serviceaccount:<namespace>:default

```
例如，如果你想要在Kube系统授予API默认账户的所有权限，你需要在规则文件中添加如下一行：
```
{"user":"system:serviceaccount:kube-system:default"}

```
重启apiserver使新添加的规则生效。

## 插件开发
其余实现的开发相对容易，API服务会调用Authorizer接口：
```sh
type Authorizer interface {
  Authorize(a Attributes) error
}
```
确认是否允许每一个API行为。

一个授权插件是实现该接口的一个模块。授权插件源码路径：pkg/auth/authorization/$MODULENAME.

一个授权模块完全是用Go语言实现的，或者可以调用一个远程授权服务。授权模块有自己的缓存，减少对相同或相似参数重复授权的成本。开发人员应该考虑缓存和权限撤销之间的相互交互。
