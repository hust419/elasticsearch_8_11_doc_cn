为了在您自己的机器上尝试Elasticsearch，我们建议使用Docker同时运行Elasticsearch和Kibana。您可以从Elastic Docker注册表中获取[Docker镜像](https://www.docker.elastic.co/)。

:::tips
从Elasticsearch 8.0开始，默认情况下启用了安全性。第一次启动Elasticsearch时，TLS加密会自动配置，为`elastic`用户生成密码，并创建一个Kibana注册令牌，以便您可以将Kibana连接到您的安全集群。
:::

有关其他安装选项，请参阅[Elasticsearch安装文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html)。

##### 启动Elasticsearch

安装并启动Docker Desktop。转到Preferences > Resources > Advanced，将Memory设置至少为4GB。
启动一个Elasticsearch容器：

```bash
docker network create elastic
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.11.1
docker run --name elasticsearch --net elastic -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -t docker.elastic.co/elasticsearch/elasticsearch:8.11.1
```

当您第一次启动Elasticsearch时，生成的`elastic`用户密码和Kibana注册令牌将输出到终端。

您可能需要在终端中向上滚动一些以查看密码和注册令牌。

复制生成的密码和注册令牌，并将它们保存在安全的位置。这些值仅在您第一次启动Elasticsearch时显示。您将使用它们来使用您的Elasticsearch集群注册Kibana并登录。

##### 启动Kibana

Kibana使您能够轻松地向Elasticsearch发送请求并以交互方式分析、可视化和管理数据。

在新的终端会话中，启动Kibana并将其连接到您的Elasticsearch容器：

```bash
docker pull docker.elastic.co/kibana/kibana:8.11.1
docker run --name kibana --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.11.1
```

启动Kibana时，会在您的终端中输出一个唯一的URL。

要访问Kibana，请在浏览器中打开生成的URL。

粘贴在启动Elasticsearch时复制的注册令牌，并单击按钮将您的Kibana实例与Elasticsearch连接。
使用在启动Elasticsearch时生成的密码登录Kibana。

##### 发送请求到Elasticsearch

通过REST API向Elasticsearch发送数据和其他请求。您可以使用任何发送HTTP请求的客户端与Elasticsearch进行交互，例如Elasticsearch语言客户端和curl。Kibana的开发者控制台提供了一个轻松的方式来实验和测试请求。要访问控制台，请转到管理 > Dev Tools。

##### 添加数据

通过REST API将JSON对象（文档）发送到Elasticsearch中，从而将数据索引到Elasticsearch中。无论您拥有结构化或非结构化文本、数值数据还是地理空间数据，Elasticsearch都会以支持快速搜索的方式有效地存储和索引它。

对于时间戳数据，例如日志和度量标准，通常将文档添加到由多个自动生成的后备索引组成的数据流中。

要将单个文档添加到索引，请提交一个目标为该索引的HTTP post请求。

```json
POST /customer/_doc/1
{
  "firstname": "Jennifer",
  "lastname": "Walters"
}
```

此请求如果该索引不存在，将自动创建customer索引，并添加一个具有ID为1的新文档，并存储和索引firstname和lastname字段。

新文档可以立即从群集中的任何节点检索。您可以使用指定其文档ID的GET请求检索它。

```json
GET /customer/_doc/1
```

要在一个请求中添加多个文档，请使用_bulk API。批量数据必须是以换行分隔的JSON（NDJSON）。每行必须以换行符（\n）结尾，包括最后一行。

```json
PUT customer/_bulk
{ "create": { } }
{ "firstname": "Monica","lastname":"Rambeau"}
{ "create": { } }
{ "firstname": "Carol","lastname":"Danvers"}
{ "create": { } }
{ "firstname": "Wanda","lastname":"Maximoff"}
{ "create": { } }
{ "firstname": "Jennifer","lastname":"Takeda"}
```

##### 搜索

索引文档在几乎实时内可用于搜索。以下搜索匹配customer索引中名为Jennifer的所有客户。

```json
GET customer/_search
{
  "query" : {
    "match" : { "firstname": "Jennifer" }
  }
}
```

##### 探索

您可以使用Kibana中的Discover来交互式地搜索和过滤数据。从那里，您可以开始创建可视化效果，并构建和共享仪表板。

要开始，请创建一个数据视图，连接到一个或多个Elasticsearch索引、数据流或索引别名。

1. 转到管理 > Stack Management > Kibana > Data Views。
2. 选择Create data view。
3. 输入数据视图的名称和一个匹配一个或多个索引的模式，例如customer。
4. 选择Save data view to Kibana。

要开始探索，转到Analytics > Discover。
