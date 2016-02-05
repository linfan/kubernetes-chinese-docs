# Labels
`译者：kz` `校对：无`




_Labels_ are key/value pairs that are attached to objects, such as pods.
Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but which do not directly imply semantics to the core system.
Labels can be used to organize and to select subsets of objects.  Labels can be attached to objects at creation time and subsequently added and modified at any time.
Each object can have a set of key/value labels defined.  Each Key must be unique for a given object.

```json
"labels": {
  "key1" : "value1",
  "key2" : "value2"
}
```

We'll eventually index and reverse-index labels for efficient queries and watches, use them to sort and group in UIs and CLIs, etc. We don't want to pollute labels with non-identifying, especially large and/or structured, data. Non-identifying information should be recorded using [annotations](annotations.md).


Labels是附加到如pod等对象的键值对。Label旨在用来指定对象那些用于辨识性目的的属性，这些属性与用户相关性大，他们来说具有意义，但是这些属性却对于不直接表达核心系统的语义（sematics）。Label可以用来组织并且选择对象的子集。Label可以在对象创建的时候依附到对象上，随后可以在任意时间被添加和修改。每一个对象可以定义一组键值对label。每一个键对于一个给定的对象必须是唯一不可重复。

```json
"labels": {
  "key1" : "value1",
  "key2" : "value2"
}
```


我们最终会索引并且反向索引（reverse-index）labels，以获得更高效的查询和监视，把他们用到UI或者CLI中用来排序或者分组等等。我们不想用那些不具有指认效果的label来污染label，特别是那些体积较大和结构型的的数据。不具有指认效果的信息应该使用annotation来记录。


## Motivation
## 目标

Labels enable users to map their own organizational structures onto system objects in a loosely coupled fashion, without requiring clients to store these mappings.

Service deployments and batch processing pipelines are often multi-dimensional entities (e.g., multiple partitions or deployments, multiple release tracks, multiple tiers, multiple micro-services per tier). Management often requires cross-cutting operations, which breaks encapsulation of strictly hierarchical representations, especially rigid hierarchies determined by the infrastructure rather than by users.

Example labels:

   * `"release" : "stable"`, `"release" : "canary"`
   * `"environment" : "dev"`, `"environment" : "qa"`, `"environment" : "production"`
   * `"tier" : "frontend"`, `"tier" : "backend"`, `"tier" : "cache"`
   * `"partition" : "customerA"`, `"partition" : "customerB"`
   * `"track" : "daily"`, `"track" : "weekly"`

These are just examples; you are free to develop your own conventions.


Label可以让用户将他们自己的有组织目的的结构以一种松耦合的方式应用到系统的对象上，且不需要客户端存放这些对应关系（mappings）。

服务部署和批处理管道通常是多维的实体（例如多个分区或者部署，多个发布轨道，多层，每层多微服务）。管理通常需要跨越式的切割操作，这会打破有严格层级展示关系的封装，特别对那些是由基础设施而非用户决定的很死板的层级关系。

一些示例label:

   * `"发行版本（release）" : "（稳定版）stable"`, `"（发行版）release" : "canary"`
   * `"环境（environment）" : "（开发）dev"`, `"environment" : "qa"`, `"environment" : "（生产）production"`
   * `"tier" : "frontend"`, `"tier" : "backend"`, `"tier" : "cache"`
   * `"分区（partition）" : "（客户A）customerA"`, `"partition" : "customerB"`
   * `"track" : "daily"`, `"track" : "weekly"`


## Syntax and character set
## 语法和字符集


_Labels_ are key value pairs. Valid label keys have two segments: an optional prefix and name, separated by a slash (`/`).  The name segment is required and must be 63 characters or less, beginning and ending with an alphanumeric character (`[a-z0-9A-Z]`) with dashes (`-`), underscores (`_`), dots (`.`), and alphanumerics between.  The prefix is optional.  If specified, the prefix must be a DNS subdomain: a series of DNS labels separated by dots (`.`), not longer than 253 characters in total, followed by a slash (`/`).
If the prefix is omitted, the label key is presumed to be private to the user. Automated system components (e.g. `kube-scheduler`, `kube-controller-manager`, `kube-apiserver`, `kubectl`, or other third-party automation) which add labels to end-user objects must specify a prefix.  The `kubernetes.io/` prefix is reserved for Kubernetes core components.

Valid label values must be 63 characters or less and must be empty or begin and end with an alphanumeric character (`[a-z0-9A-Z]`) with dashes (`-`), underscores (`_`), dots (`.`), and alphanumerics between.


Labels是键值对。合法的键值对有两个部分：一个可选的前缀和名字，由“/”分隔。名字部分是必须，长度一定是64个字符或者更短，首位字符必须是字母数字型（`[a-z0-9A-Z]`），其他地方的字符必须是横线，下划线，点，或者其他字母数字字符。前缀是可选的。如果没有指定，前缀必须是一个dns的子域：一系列有点分隔的DNS label，总长度不能超过253个字符，以下划线“/”结尾。如果前缀被省略掉了，label的键会被认为是对于用户私有的。系统自动的组件（如kube-scheduler，kube-controller-manager，kube-apiserver，kubectl或者其他第三方自动工具）如要向用户最终的对象添加label，必须指定前缀。前缀`kubernetes.io/`是保留给Kubernetes核心组件用的。

合法的label值必须是63个或者更短的字符。要么是空，要么首位字符必须为字母数字字符，中间必须是横线，下划线，点或者数字字母。



## Label选择器

Unlike [names and UIDs](identifiers.md), labels do not provide uniqueness. In general, we expect many objects to carry the same label(s).

Via a _label selector_, the client/user can identify a set of objects. The label selector is the core grouping primitive in Kubernetes.

The API currently supports two types of selectors: _equality-based_ and _set-based_.
A label selector can be made of multiple _requirements_ which are comma-separated. In the case of multiple requirements, all must be satisfied so the comma separator acts as an _AND_ logical operator.

An empty label selector (that is, one with zero requirements) selects every object in the collection.

A null label selector (which is only possible for optional selector fields) selects no objects.

与name和UID不同，label不提供唯一性。通常，我们会看到很多对象有着一样的label。

通过label选择器，客户端/用户能方便辨识出一组对象。label选择器是kubernetes中核心的组织原语。

API目前支持两种选择器：基于相等的和基于集合的。一个label选择器一可以由多个必须条件组成，由逗号分隔。在多个必须条件指定的情况下，所有的条件都必须满足，因而逗号起着AND逻辑运算符的作用。

一个空的label选择器（即有0个必须条件的选择器）会选择集合中的每一个对象。

 一个null型label选择器（仅对于可选的选择器字段才可能）不会返回任何对象。




### _Equality-based_ requirement
### 基于相等性的条件

_Equality-_ or _inequality-based_ requirements allow filtering by label keys and values. Matching objects must satisfy all of the specified label constraints, though they may have additional labels as well.
Three kinds of operators are admitted `=`,`==`,`!=`. The first two represent _equality_ (and are simply synonyms), while the latter represents _inequality_. For example:


```
environment = production
tier != frontend
```

The former selects all resources with key equal to `environment` and value equal to `production`.
The latter selects all resources with key equal to `tier` and value distinct from `frontend`, and all resources with no labels with the `tier` key.
One could filter for resources in `production` excluding `frontend` using the comma operator: `environment=production,tier!=frontend`

基于相等性或者不相等性的条件允许用label的键或者值进行过滤。匹配的对象必须满足所有指定的label约束，尽管他们可能也有额外的label。有三种运算符是允许的，“=”，“==”和“!=”。前两种代表相等性（他们是同义运算符），后一种代表非相等性。例如：

```
environment = production
tier != frontend
```

第一个选择所有键等于`environment`值为`production`的资源。后一种选择所有键为`tier`值不等于`frontend`的资源，和那些没有键为`tier`的label的资源。

要过滤所有处于`production`但不是`frontend`的资源，可以使用逗号操作符，`environment=production,tier!=frontend`。


### 基于set的条件

_Set-based_ label requirements allow filtering keys according to a set of values. Three kinds of operators are supported: `in`,`notin` and exists (only the key identifier). For example:

```
environment in (production, qa)
tier notin (frontend, backend)
partition
!partition
```

The first example selects all resources with key equal to `environment` and value equal to `production` or `qa`.
The second example selects all resources with key equal to `tier` and values other than `frontend` and `backend`, and all resources with no labels with the `tier` key.
The third example selects all resources including a label with key `partition`; no values are checked.
The fourth example selects all resources without a label with key `partition`; no values are checked.
Similarly the comma separator acts as an _AND_ operator. So filtering resources with a `partition` key (no matter the value) and with `environment` different than  `qa` can be achieved using `partition,environment notin (qa)`.
The _set-based_ label selector is a general form of equality since `environment=production` is equivalent to `environment in (production)`; similarly for `!=` and `notin`.

_Set-based_ requirements can be mixed with _equality-based_ requirements. For example: `partition in (customerA, customerB),environment!=qa`.


基于集合的label条件允许用一组值来过滤键。支持三种操作符:`in`，`notin`,和`exists(仅针对于key符号)`。例如：

```
environment in (production, qa)
tier notin (frontend, backend)
partition
!partitio

```

第一个例子，选择所有键等于`environment`，且value等于`production`或者`qa`的资源。
第二个例子，选择所有键等于`tier`且值是除了`frontend`和`backend`之外的资源，和那些没有label的键是`tier`的资源。
第三个例子，选择所有所有有一个label的键为partition的资源；值是什么不会被检查。
第四个例子，选择所有的没有lable的键名为`partition`的资源；值是什么不会被检查。

类似的，逗号操作符相当于一个__AND__操作符。因而要使用一个`partition`键（不管值是什么），并且`environment`不是`qa`过滤资源可以用`partition,environment notin (qa)`。

基于集合的选择器是一个相等性的宽泛的形式，因为`environment=production`相当于 `environment in (production)`，与`!=` and `notin`类似。

基于集合的条件可以与基于相等性 的条件混合。例如，`partition in (customerA, customerB),environment!=qa`。


## API

### LIST and WATCH filtering
### LIST 和WATCH过滤

LIST and WATCH operations may specify label selectors to filter the sets of objects returned using a query parameter. Both requirements are permitted:

    * _equality-based_ requirements: `?labelSelector=environment%3Dproduction,tier%3Dfrontend`
    * _set-based_ requirements: `?labelSelector=environment+in+%28production%2Cqa%29%2Ctier+in+%28frontend%29`

LIST和WATCH操作，可以使用query参数来指定label选择器来过滤返回对象的集合。两种条件都可以使用：
	*基于相等性条件：`?labelSelector=environment%3Dproduction,tier%3Dfrontend`
	*基于集合条件的：`?labelSelector=environment+in+%28production%2Cqa%29%2Ctier+in+%28frontend%29`


Both label selector styles can be used to list or watch resources via a REST client. For example targetting `apiserver` with `kubectl` and using _equality-based_ one may write:


两种label选择器风格都可以用来通过REST客户端来列表或者监视资源。比如使用`kubectl`来针对`apiserver`，并且使用基于相等性的条件，可以用：

·
```console
$ kubectl get pods -l environment=production,tier=frontend
```

or using _set-based_ requirements:
或者使用基于集合的条件：

```console
$ kubectl get pods -l 'environment in (production),tier in (frontend)'
```

As already mentioned _set-based_ requirements are more expressive.  For instance, they can implement the _OR_ operator on values:

如以上已经提到的，基于集合的条件表达性更强。例如，他们可以实现值上的OR操作：

```console
$ kubectl get pods -l 'environment in (production, qa)'
```

or restricting negative matching via _exists_ operator:

或者通过exists操作符进行否定限制匹配：

```console
$ kubectl get pods -l 'environment,environment notin (frontend)'
```



### Set references in API objects

Some Kubernetes objects, such as [`service`s](services.md) and [`replicationcontroller`s](replication-controller.md), also use label selectors to specify sets of other resources, such as [pods](pods.md).

一些Kubernetes对象，比如service和replication controlle的，也使用label选择器来指定其他资源的集合，比如pods。

#### Service and ReplicationController

The set of pods that a `service` targets is defined with a label selector. Similarly, the population of pods that a `replicationcontroller` should manage is also defined with a label selector.

Labels selectors for both objects are defined in `json` or `yaml` files using maps, and only _equality-based_ requirement selectors are supported:

```json
"selector": {
    "component" : "redis",
}
```

or

```yaml
selector:
    component: redis
```

this selector (respectively in `json` or `yaml` format) is equivalent to `component=redis` or `component in (redis)`.


一个service针对的pods的集合是用label选择器来定义的。类似的，一个replicationcontroller管理的pods的群体也是用label选择器来定义的。

对于这两种对象的Label选择器是用map定义在json或者yaml文件中的，并且只支持基于相等性的条件：



```json
"selector": {
    "component" : "redis",
}
```

或者

```yaml
selector:
    component: redis
```


这个选择器（分别是位于json或者yaml格式的）相等于`component=redis` 或者 `component in (redis)`。


#### Job and other new resources

### Job和其他新的资源


Newer resources, such as [job](jobs.md), support _set-based_ requirements as well.

```yaml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```

`matchLabels` is a map of `{key,value}` pairs. A single `{key,value}` in the `matchLabels` map is equivalent to an element of `matchExpressions`, whose `key` field is "key", the `operator` is "In", and the `values` array contains only "value". `matchExpressions` is a list of pod selector requirements. Valid operators include In, NotIn, Exists, and DoesNotExist. The values set must be non-empty in the case of In and NotIn. All of the requirements, from both `matchLabels` and `matchExpressions` are ANDed together -- they must all be satisfied in order to match.


较新的资源，如job，也支持基于集合的条件。

```yaml
selector:
  matchLabels:相当于一个
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```
`matchLabels`是一个键值对的映射。一个单独的`{key,value}`相当于`matchExpressions`的一个元素，它的键字段是"key",操作符是`In`，并且值数组值包含"value"。`matchExpressions`是一个pod的选择器条件的列表。合法的操作符包含In, NotIn, Exists, and DoesNotExist。在In和NotIn的情况下，值的组必须不能为空。所有的条件，包含`matchLabels` and `matchExpressions`中的，会用AND符号连接，他们必须都被满足以完成匹配。
