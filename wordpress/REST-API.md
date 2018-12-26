# REST API

https://app.jangrui.com/wp-json/

|Resource	    |接口	|Base Route |
|-              |- 		|-          |
|Posts	        |文章 	|/wp/v2/posts
|Post Revisions	|文章修订	|/wp/v2/revisions
|Categories	    |分类	|/wp/v2/categories
|Tags	        |标签	|/wp/v2/tags
|Pages	        |固定页面	|/wp/v2/pages
|Comments	    |评论	|/wp/v2/comments
|Taxonomies	    |       |/wp/v2/taxonomies
|Media	        |媒体	|/wp/v2/media
|Users	        |用户	|/wp/v2/users
|Post Types	    |文章类型	|/wp/v2/types
|Post Statuses	|文章状态	|/wp/v2/statuses
|Settings	    |设置	|/wp/v2/settings


## posts:

|参数	|类型	|说明|动作
|-|-|-|-|
| date 	| string|对象发布的日期，在站点的时区中	|view，edit，embed
|date_gmt|string|对象发布的日期，格林威治标准时间	|view，edit
|guid	|object |对象的全局唯一标识符。只读		|view，edit
|id 	|integer|对象的唯一标识符。只读 			|view，edit，embed
|link 	|string、uri| 	对象的URL。只读		|view，edit，embed
|modified|string，datetime|上次修改对象的日期，位于站点的时区中。只读|view，edit
|modified_gmt|string，datetime|上次修改对象的日期，格林尼治标准时间。只读|view，edit
|slug	|string |对象类型唯一的对象的字母数字标识符。|view，edit，embed
|status| string|对象的命名状态。|view，edit
|type|string|对象的帖子类型。只读|view，edit，embed
|password|string|用于保护对内容和摘录的访问的密码。|edit
|title|object|对象的标题。|view，edit，embed
|content|object|对象的内容。|view，edit
|author|integer|对象作者的ID。|view，edit，embed
|excerpt|object|对象的摘录。|view，edit，embed
|featured_media|integer|对象的特色媒体的ID。|view，edit，embed
|comment_status|string|是否对对象打开注释。|view，edit
|ping_status|string|是否可以ping通对象。|view，edit
|format|string|对象的格式。|view，edit
|meta|object|元字段。|view，edit
|sticky|boolean|是否应将对象视为粘性。|view，edit
|template|string|用于显示对象的主题文件。|view，edit
|categories|array|分配给类别分类中对象的术语。|view，edit
|tags|array|在post_tag分类中分配给对象的术语。|view，edit

### 示例请求

$ curl -X OPTIONS -i http://demo.wp-api.org/wp-json/wp/v2/posts

### 列表帖子

#### 参数

>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)
>- page	该系列的当前页面。默认： 1
>- per_page	结果集中要返回的最大项目数。默认： 10
>- search	将结果限制为匹配字符串的结果。
>- after	限制对符合ISO8601标准的日期之后发布的帖子的响应。
>- author	将结果集限制为分配给特定作者的帖子。
>- author_exclude	确保结果集排除分配给特定作者的帖子。
>- before	限制对在给定的ISO8601合规日期之前发布的帖子的响应。
>- exclude	确保结果集排除特定ID。
>- include	将结果集限制为特定ID。
>- offset	通过特定数量的项目抵消结果集。
>- order	订单排序属性升序或降序。默认： desc (asc，desc)
>- orderby	按对象属性排序集合。默认： date (author，date，id，include，modified，parent，relevance，slug，title)
>- slug	将结果集限制为具有一个或多个特定slug的帖子。
>- status	将结果集限制为分配了一个或多个状态的帖子。默认： publish
>- categories	将结果集限制为在类别分类中指定了指定术语的所有项目。
>- categories_exclude	将结果集限制为除了在类别分类中指定了指定术语的项目之外的所有项目。
>- tags	将结果集限制为在标签分类中指定了指定术语的所有项目。
>- tags_exclude	将结果集限制为除标签分类中指定了术语的项目之外的所有项目。
>- sticky	将结果集限制为粘性项。

#### 定义

GET /wp/v2/posts

### 示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/posts

### 创建帖子

#### 参数

>- date	对象发布的日期，在站点的时区中。
>- date_gmt	对象发布的日期，格林威治标准时间。
>- slug	对象类型唯一的对象的字母数字标识符。
>- status	对象的命名状态。(publish，future，draft，pending，private)
>- password	用于保护对内容和摘录的访问的密码。
>- title	对象的标题。
>- content	对象的内容。
>- author	对象作者的ID。
>- excerpt	对象的摘录。
>- featured_media	对象的特色媒体的ID。
>- comment_status	是否对对象打开注释。(open，closed)
>- ping_status	是否可以ping通对象。(open，closed)
>- format	对象的格式。(standard，aside，chat，gallery，link，image，quote，status，video，audio)
>- meta	元字段。
>- sticky	是否应将对象视为粘性。
>- template	用于显示对象的主题文件。
>- categories	分配给类别分类中对象的术语。
>- tags	在post_tag分类中分配给对象的术语。

#### 定义

POST /wp/v2/posts

检索帖子

#### 参数

>- id	对象的唯一标识符。
>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)
>- password	如果邮件受密码保护，则为密码。

#### 定义

GET /wp/v2/posts/<id>

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/posts/<id>

更新帖子

#### 参数

>- id	对象的唯一标识符。
>- date	对象发布的日期，在站点的时区中。
>- date_gmt	对象发布的日期，格林威治标准时间。
>- slug	对象类型唯一的对象的字母数字标识符。
>- status	对象的命名状态。(publish，future，draft，pending，private)
>- password	用于保护对内容和摘录的访问的密码。
>- title	对象的标题。
>- content	对象的内容。
>- author	对象作者的ID。
>- excerpt	对象的摘录。
>- featured_media	对象的特色媒体的ID。
>- comment_status	是否对对象打开注释。(open，closed)
>- ping_status	是否可以ping通对象。(open，closed)
>- format	对象的格式。(standard，aside，chat，gallery，link，image，quote，status，video，audio)
>- meta	元字段。
>- sticky	是否应将对象视为粘性。
>- template	用于显示对象的主题文件。
>- categories	分配给类别分类中对象的术语。
>- tags	在post_tag分类中分配给对象的术语。

#### 定义

POST /wp/v2/posts/<id>

示例请求

$ curl -X POST http://demo.wp-api.org/wp-json/wp/v2/posts/<id> -d '{"title":"My New Title"}'

删除帖子

#### 参数

>- id	对象的唯一标识符。
>- force	是否绕过垃圾并强行删除。

#### 定义

DELETE /wp/v2/posts/<id>

示例请求

$ curl -X DELETE http://demo.wp-api.org/wp-json/wp/v2/posts/<id>

## 文章修订 Post Revisions：

|参数|类型|说明|动作
|-|-|-|-|
|author|integer|对象作者的ID|view，edit，embed
|date|string，datetime[详情](https://core.trac.wordpress.org/ticket/41032)|对象发布的日期，在站点的时区中|view，edit，embed
|date_gmt|string，datetime [详情](https://core.trac.wordpress.org/ticket/41032)|对象发布的日期，格林威治标准时间|view，edit
|guid|object|对象的全局唯一标识符 只读|view，edit
|id|integer|对象的唯一标识符|view，edit，embed
|modified|string,datetime|上次修改对象的日期，位于站点的时区中|view，edit
|modified_gmt|string，datetime|上次修改对象的日期，格林尼治标准时间|view，edit
|parent|integer|对象父级的ID||view，edit，embed
|slug|string|对象类型唯一的对象的字母数字标识符|view，edit，embed
|title|object|对象的标题|view，edit，embed
|content|object|对象的内容|view，edit
|excerpt|object|对象的摘录|view，edit，embed

示例请求
$ curl -X OPTIONS -i http://demo.wp-api.org/wp-json/wp/v2/posts/<parent>/revisions

检索修订后的内容

#### 参数

>- parent	对象父级的ID。
>- context	提出请求的范围; 确定响应中存在的字段。

默认： view (view，embed，edit)

#### 定义

GET /wp/v2/posts/<parent>/revisions

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/posts/<parent>/revisions

检索修订后的内容

#### 参数

>- parent	对象父级的ID。
>- id	对象的唯一标识符。
>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)

#### 定义

GET /wp/v2/posts/<parent>/revisions/<id>

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/posts/<parent>/revisions/<id>

删除修订后的内容

#### 参数

>- parent	对象父级的ID。
>- id	对象的唯一标识符。
>- force	要求为真，因为修订不支持废弃。

#### 定义

DELETE /wp/v2/posts/<parent>/revisions/<id>

示例请求

$ curl -X DELETE http://demo.wp-api.org/wp-json/wp/v2/posts/<parent>/revisions/<id>

## Categories :

|||||
|-|-|-|-|
|id|整数|该术语的唯一标识符 只读|view，embed，edit
|count|整数|该术语的已发布帖子数 只读|view，edit
|description|串|该术语的HTML描述|view，edit
|link|字符串，uri|该术语的URL 只读|view，embed，edit
|name|串	|该术语的HTML标题|view，embed，edit
|slug|串	|该类型唯一的术语的字母数字标识符。|view，embed，edit
|taxonomy|串|输入该字词的归因 只读|view，embed，edit （--category，post_tag，nav_menu，link_category，post_format）
|parent|整数|父条款ID|view，edit
|meta|宾语|元字段||view，edit

示例请求

$ curl -X OPTIONS -i http://demo.wp-api.org/wp-json/wp/v2/categories

列表类别

#### 参数

>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)
>- page	该系列的当前页面。默认： 1
>- per_page	结果集中要返回的最大项目数。默认： 10
>- search	将结果限制为匹配字符串的结果。
>- exclude	确保结果集排除特定ID。
>- include	将结果集限制为特定ID。
>- order	订单排序属性升序或降序。默认： asc (asc，desc)
>- orderby	按术语属性排序集合。默认： name (id，include，name，slug，term_group，description，count)
>- hide_empty	是否隐藏未分配给任何帖子的条款。
>- parent	将结果集限制为分配给特定父级的术语。
>- post	将结果集限制为分配给特定帖子的术语。
>- slug	将结果集限制为具有一个或多个特定段的术语。

#### 定义

GET /wp/v2/categories

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/categories

创建类别

#### 参数

>- description	该术语的HTML描述。
>- name	该术语的HTML标题。要求：1
>- slug	该类型唯一的术语的字母数字标识符。
>- parent	父条款ID。
>- meta	元字段。

#### 定义

POST /wp/v2/categories

检索类别

#### 参数

> - id	该术语的唯一标识符。
> - context	提出请求的范围; 确定响应中存在的字段。view (view，embed，edit)

#### 定义

GET /wp/v2/categories/<id>

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/categories/<id>

更新类别

#### 参数

>- id	该术语的唯一标识符。
>- description	该术语的HTML描述。
>- name	该术语的HTML标题。
>- slug	该类型唯一的术语的字母数字标识符。
>- parent	父条款ID。
>- meta	元字段。

#### 定义

POST /wp/v2/categories/<id>

删除类别

#### 参数

>- id	该术语的唯一标识符。
>- force	要求为真，因为条款不支持废弃。

#### 定义

DELETE /wp/v2/categories/<id>

示例请求

$ curl -X DELETE http://demo.wp-api.org/wp-json/wp/v2/categories/<id>

## Tags:

### Schema

|参数|类型|说明|动作|
|-|-|-|-|
|id|整数|该术语的唯一标识符 只读|view，embed，edit
|count|整数|该术语的已发布帖子数 只读|view，edit
|description|串|该术语的HTML描述|view，edit
|link|字符串，uri|该术语的URL 只读|view，embed，edit
|name|串|该术语的HTML标题|view，embed，edit
|slug|串|该类型唯一的术语的字母数字标识符|view，embed，edit
|taxonomy|串|输入该字词的归因 只读|view，embed，edit(一：category，post_tag，nav_menu，link_category，post_format)
|meta|宾语|元字段|view，edit

### 列出标签

#### 参数：

>- context	提出请求的范围; 确定响应中存在的字段。
>- page	该系列的当前页面。
>- per_page	结果集中要返回的最大项目数。默认：10
>- search	将结果限制为匹配字符串的结果。
>- exclude	确保结果集排除特定ID。
>- include	将结果集限制为特定ID。
>- offset	通过特定数量的项目抵消结果集。
>- order	订单排序属性升序或降序。默认： asc (asc，desc)
>- orderby	按术语属性排序集合。默认：name (id，include，name，slug，term_group，description，count)
>- hide_empty	是否隐藏未分配给任何帖子的条款。
>- post	将结果集限制为分配给特定帖子的术语。
>- slug	将结果集限制为具有一个或多个特定段的术语。

#### 定义

GET /wp/v2/tags

### 创建标签

#### 参数

>- description	该术语的HTML描述。
>- name	该术语的HTML标题。
>- slug	该类型唯一的术语的字母数字标识符。
>- meta	元字段。

#### 定义

POST /wp/v2/tags

### 检索标签

#### 参数

>- id	该术语的唯一标识符。
>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)

#### 定义

GET /wp/v2/tags/<id>

### 更新标签

#### 参数

>s- id	该术语的唯一标识符。
>s- description	该术语的HTML描述。
>s- name	该术语的HTML标题。
>s- slug	该类型唯一的术语的字母数字标识符。
>s- meta	元字段。

#### 定义

POST /wp/v2/tags/<id>

### 删除标签

#### 参数

>- id	该术语的唯一标识符。
>- force	要求为真，因为条款不支持废弃。

#### 定义

DELETE /wp/v2/tags/<id>



## pages 页面

### 列表页面

#### 参数

>- context	提出请求的范围; 确定响应中存在的字段。默认：view (view，embed，edit)
>- page	该系列的当前页面。
>- per_page	结果集中要返回的最大项目数。默认：10
>- search	将结果限制为匹配字符串的结果。
>- after	限制对符合ISO8601标准的日期之后发布的帖子的响应。
>- author	将结果集限制为分配给特定作者的帖子。
>- author_exclude	确保结果集排除分配给特定作者的帖子。
>- before	限制对在给定的ISO8601合规日期之前发布的帖子的响应。
>- exclude	确保结果集排除特定ID。
>- include	将结果集限制为特定ID。
>- menu_order	将结果集限制为具有特定menu_order值的帖子。
>- offset	通过特定数量的项目抵消结果集。
>- order	订单排序属性升序或降序。默认： desc (asc，desc)
>- orderby	按对象属性排序集合。默认: date (author，date，id，include，modified，parent，relevance，slug，title，menu_order)
>- parent	将结果集限制为具有特定父ID的项目。
>- parent_exclude	将结果集限制为除特定父ID之外的所有项目。
>- slug	将结果集限制为具有一个或多个特定slug的帖子。
>- status	将结果集限制为分配了一个或多个状态的帖子。默认： publish

#### 定义

GET /wp/v2/pages

#### 示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/pages

### 创建页面

#### 参数

>- date	对象发布的日期，在站点的时区中。
>- date_gmt	对象发布的日期，格林威治标准时间。
>- slug	对象类型唯一的对象的字母数字标识符。
>- status	对象的命名状态。(publish，future，draft，pending，private)
>- password	用于保护对内容和摘录的访问的密码。
>- parent	对象父级的ID。
>- title	对象的标题。
>- content	对象的内容。
>- author	对象作者的ID。
>- excerpt	对象的摘录。
>- featured_media	对象的特色媒体的ID。
>- comment_status	是否对对象打开注释。(open，closed)
>- ping_status	是否可以ping通对象。(open，closed)
>- menu_order	对象相对于其类型的其他对象的顺序。
>- meta	元字段。
>- template	用于显示对象的主题文件。

#### 定义

POST /wp/v2/pages

### 检索页面

#### 参数

>- id	对象的唯一标识符。
>- context	提出请求的范围; 确定响应中存在的字段。
>- password	如果邮件受密码保护，则为密码。


#### 定义

GET /wp/v2/pages/<id>

#### 示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/pages/<id>

### 更新页面

#### 参数

>- id	对象的唯一标识符。
>- date	对象发布的日期，在站点的时区中。
>- date_gmt	对象发布的日期，格林威治标准时间。
>- slug	对象类型唯一的对象的字母数字标识符。
>- status	对象的命名状态。(publish，future，draft，pending，private)
>- password	用于保护对内容和摘录的访问的密码。
>- parent	对象父级的ID。
>- title	对象的标题。
>- content	对象的内容。
>- author	对象作者的ID。
>- excerpt	对象的摘录。
>- featured_media	对象的特色媒体的ID。
>- comment_status	是否对对象打开注释。(open，closed)
>- ping_status	是否可以ping通对象。(open，closed)
>- menu_order	对象相对于其类型的其他对象的顺序。
>- meta	元字段。
>- template	用于显示对象的主题文件。

#### 定义

POST /wp/v2/pages/<id>


### 删除页面

#### 参数

>- id	对象的唯一标识符。
>- force	是否绕过垃圾并强行删除。

#### 定义

DELETE /wp/v2/pages/<id>

#### 示例请求

$ curl -X DELETE http://demo.wp-api.org/wp-json/wp/v2/pages/<id>

## 评论 comments

###列表评论

#### 参数

>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)
>- page	该系列的当前页面。
>- per_page	结果集中要返回的最大项目数。默认： 10
>- search	将结果限制为匹配字符串的结果。
>- after	限制对符合ISO8601标准的日期之后发布的评论的响应。
>- author	将结果集限制为分配给特定用户ID的注释。需要授权。
>- author_exclude	确保结果集排除分配给特定用户ID的注释。需要授权。
>- author_email	将结果集限制为特定作者电子邮件的结果集。需要授权。
>- before	限制对符合ISO8601标准的日期之前发布的评论的响应。
>- exclude	确保结果集排除特定ID。
>- include	将结果集限制为特定ID。
>- offset	通过特定数量的项目抵消结果集。
>- order	订单排序属性升序或降序。默认： desc (asc，desc)
>- orderby	按对象属性排序集合。默认： date_gmt (date，date_gmt，id，include，post，parent，type)
>- parent	将结果集限制为特定父ID的注释。
>- parent_exclude	确保结果集排除特定的父ID。
>- post	将结果集限制为分配给特定帖子ID的注释。
>- status	将结果集限制为分配了特定状态的注释。需要授权。默认： approve
>- type	将结果集限制为指定特定类型的注释。需要授权。默认： comment
>- password	如果邮件受密码保护，则为密码。

#### 定义

GET /wp/v2/comments

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/comments

### 创建评论

#### 参数

>- author	如果作者是用户，则为用户对象的ID。
>- author_email	对象作者的电子邮件地址。
>- author_ip	对象作者的IP地址。
>- author_name	显示对象作者的名称。
>- author_url	对象作者的URL。
>- author_user_agent	对象作者的用户代理。
>- content	对象的内容。
>- date	对象发布的日期，在站点的时区中。
>- date_gmt	对象发布的日期，格林威治标准时间。
>- parent	对象父级的ID。
>- post	关联的帖子对象的ID。
>- status	对象的状态。
>- meta	元字段。

#### 定义

POST /wp/v2/comments

### 检索评论

#### 参数

>- id	对象的唯一标识符。
>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)
>- password	注释的父帖子的密码（如果帖子受密码保护）。

#### 定义

GET /wp/v2/comments/<id>

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/comments/<id>


### 更新评论

#### 参数

> - id	对象的唯一标识符。
> - author	如果作者是用户，则为用户对象的ID。
> - author_email	对象作者的电子邮件地址。
> - author_ip	对象作者的IP地址。
> - author_name	显示对象作者的名称。
> - author_url	对象作者的URL。
> - author_user_agent	对象作者的用户代理。
> - content	对象的内容。
> - date	对象发布的日期，在站点的时区中。
> - date_gmt	对象发布的日期，格林威治标准时间。
> - parent	对象父级的ID。
> - post	关联的帖子对象的ID。
> - status	对象的状态。
> - meta	元字段。

#### 定义

POST /wp/v2/comments/<id>

### 删除评论

#### 参数

> id	对象的唯一标识符。
> force	是否绕过垃圾并强行删除。
> password	注释的父帖子的密码（如果帖子受密码保护）。

#### 定义

DELETE /wp/v2/comments/<id>

示例请求

$ curl -X DELETE http://demo.wp-api.org/wp-json/wp/v2/comments/<id>

## 媒体 media

### 列出媒体

#### 参数

> - context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)
> - page	该系列的当前页面。
> - per_page	结果集中要返回的最大项目数。默认： 10
> - search	将结果限制为匹配字符串的结果。
> - after	限制对符合ISO8601标准的日期之后发布的帖子的响应。
> - author	将结果集限制为分配给特定作者的帖子。
> - author_exclude	确保结果集排除分配给特定作者的帖子。
> - before	限制对在给定的ISO8601合规日期之前发布的帖子的响应。
> - exclude	确保结果集排除特定ID。
> - include	将结果集限制为特定ID。
> - offset	通过特定数量的项目抵消结果集。
> - order	订单排序属性升序或降序。默认： desc (asc，desc)
> - orderby	按对象属性排序集合。默认： date (author，date，id，include，modified，parent，relevance，slug，title)
> - parent	将结果集限制为具有特定父ID的项目。
> - parent_exclude	将结果集限制为除特定父ID之外的所有项目。
> - slug	将结果集限制为具有一个或多个特定slug的帖子。
> - status	将结果集限制为分配了一个或多个状态的帖子。默认： inherit
> - media_type	将结果集限制为特定媒体类型的附件。(image，video，audio，application)
> - mime_type	将结果集限制为特定MIME类型的附件。


#### 定义

GET /wp/v2/media

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/media

创建媒体项目

#### 参数

> - date	对象发布的日期，在站点的时区中。
> - date_gmt	对象发布的日期，格林威治标准时间。
> - slug	对象类型唯一的对象的字母数字标识符。
> - status	对象的命名状态。(publish，future，draft，pending，private)
> - title	对象的标题。
> - author	对象作者的ID。
> - comment_status	是否对对象打开注释。(open，closed)
> - ping_status	是否可以ping通对象。(open，closed)
> - meta	元字段。
> - template	用于显示对象的主题文件。
> - alt_text	未显示附件时显示的替代文本。
> - caption	附件标题。
> - description	附件说明。
> - post	附件的相关帖子的ID。

#### 定义

POST /wp/v2/media

检索媒体项目

#### 参数

> - id	对象的唯一标识符。
> - context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)

#### 定义

GET /wp/v2/media/<id>

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/media/<id>

更新媒体项目

#### 参数

> - id	对象的唯一标识符。
> - date	对象发布的日期，在站点的时区中。
> - date_gmt	对象发布的日期，格林威治标准时间。
> - slug	对象类型唯一的对象的字母数字标识符。
> - status	对象的命名状态。(publish，future，draft，pending，private)
> - title	对象的标题。
> - author	对象作者的ID。
> - comment_status	是否对对象打开注释。(open，closed)
> - ping_status	是否可以ping通对象。(open，closed)
> - meta	元字段。
> - template	用于显示对象的主题文件。
> - alt_text	未显示附件时显示的替代文本。
> - caption	附件标题。
> - description	附件说明。
> - post	附件的相关帖子的ID。

#### 定义

POST /wp/v2/media/<id>

示例请求

删除媒体项目

#### 参数

> - id	对象的唯一标识符。
> - force	是否绕过垃圾并强行删除。

#### 定义

DELETE /wp/v2/media/<id>

示例请求

$ curl -X DELETE http://demo.wp-api.org/wp-json/wp/v2/media/<id>

## 用户

列出用户


#### 参数

> - context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)
> - page	该系列的当前页面.
> - per_page	结果集中要返回的最大项目数。默认： 10
> - search	将结果限制为匹配字符串的结果。
> - exclude	确保结果集排除特定ID。
> - include	将结果集限制为特定ID。
> - offset	通过特定数量的项目抵消结果集。
> - order	订单排序属性升序或降序。默认： asc (asc，desc)
> - orderby	按对象属性排序集合。默认： name (id，include，name，registered_date，slug，email，url)
> - slug	将结果集限制为具有一个或多个特定slug的用户。
> - roles	将结果集限制为与至少一个提供的特定角色匹配的用户。接受csv列表或单个角色。

#### 定义

GET /wp/v2/users

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/users

创建用户

#### 参数

> - username	用户的登录名。
> - name	显示用户的名称。
> - first_name	用户的名字。
> - last_name	用户的姓氏。
> - email	用户的电子邮件地址。
> - url	用户的URL。
> - description	用户描述。
> - locale	用户的区域设置。en_US
> - nickname	用户的昵称。
> - slug	用户的字母数字标识符。
> - roles	分配给用户的角色。
> - password	用户密码（从未包括在内）。
> - meta	元字段。

#### 定义

POST /wp/v2/users

检索用户

#### 参数

>- id	用户的唯一标识符。
>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)

#### 定义

GET /wp/v2/users/<id>

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/users/<id>

更新用户

#### 参数

>- id	用户的唯一标识符。
>- username	用户的登录名。
>- name	显示用户的名称。
>- first_name	用户的名字。
>- last_name	用户的姓氏。
>- email	用户的电子邮件地址。
>- url	用户的URL。
>- description	用户描述。
>- locale	用户的区域设置。en_US
>- nickname	用户的昵称。
>- slug	用户的字母数字标识符。
>- roles	分配给用户的角色。
>- password	用户密码（从未包括在内）。
>- meta	元字段。

#### 定义

POST /wp/v2/users/<id>

删除用户

#### 参数

>- id	用户的唯一标识符。
>- force	要求为真，因为用户不支持废弃。
>- reassign	将已删除用户的帖子和链接重新分配给此用户ID。

#### 定义

DELETE /wp/v2/users/<id>

示例请求

$ curl -X DELETE http://demo.wp-api.org/wp-json/wp/v2/users/<id>

检索用户


#### 参数

>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)

#### 定义

GET /wp/v2/users/me

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/users/me

更新用户

#### 参数

>- username	用户的登录名。
>- name	显示用户的名称。
>- first_name	用户的名字。
>- last_name	用户的姓氏。
>- email	用户的电子邮件地址。
>- url	用户的URL。
>- description	用户描述。
>- locale	用户的区域设置。en_US
>- nickname	用户的昵称。
>- slug	用户的字母数字标识符。
>- roles	分配给用户的角色。
>- password	用户密码（从未包括在内）。
>- meta	元字段。

#### 定义

POST /wp/v2/users/me

删除用户

#### 参数

>- force	要求为真，因为用户不支持废弃。
>- reassign	将已删除用户的帖子和链接重新分配给此用户ID。

#### 定义

DELETE /wp/v2/users/me

示例请求

$ curl -X DELETE http://demo.wp-api.org/wp-json/wp/v2/users/me


## 设置 settings

示例请求

$ curl -X OPTIONS -i http://demo.wp-api.org/wp-json/wp/v2/settings

检索设置

#### 定义

GET /wp/v2/settings

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/settings

更新设置

#### 参数

>- title	网站标题。
>- description	网站标语。
>- timezone	与您在同一时区的城市。
>- date_format	所有日期字符串的日期格式。
>- time_format	所有时间字符串的时间格式。
>- start_of_week	本周开始的一周中的一天数。
>- language	WordPress区域设置代码。
>- use_smilies	将表情符号:-)和：-P转换为显示的图形。
>- default_category	默认帖子类别。
>- default_post_format	默认发布格式。
>- posts_per_page	博客页面最多显示。
>- default_ping_status	允许来自其他博客（pingback和引用）的新文章的链接通知。(open，closed)
>- default_comment_status	允许人们对新文章发表评论。(open，closed)

#### 定义

POST /wp/v2/settings

## 文章类型 post-types

示例请求

$ curl -X OPTIONS -i http://demo.wp-api.org/wp-json/wp/v2/types

检索类型

#### 参数

>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)

#### 定义

GET /wp/v2/types

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/types

检索类型

#### 参数

>- type	邮政类型的字母数字标识符。
>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)

#### 定义

GET /wp/v2/types/<type>

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/types/<type>

## 文章状态 post-statuses

示例请求

$ curl -X OPTIONS -i http://demo.wp-api.org/wp-json/wp/v2/statuses

检索状态

#### 参数

>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)

#### 定义

GET /wp/v2/statuses

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/statuses

检索状态

#### 参数

>- status	状态的字母数字标识符。
>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)

#### 定义

GET /wp/v2/statuses/<status>

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/statuses/<status>

## 索引 taxonomies

示例请求

$ curl -X OPTIONS -i http://demo.wp-api.org/wp-json/wp/v2/taxonomies

检索分类法

#### 参数

>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)
>- type	将结果限制为与特定帖子类型相关联的分类。

#### 定义

GET /wp/v2/taxonomies

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/taxonomies

检索分类法

#### 参数

>- taxonomy	分类的字母数字标识符。
>- context	提出请求的范围; 确定响应中存在的字段。默认： view (view，embed，edit)

#### 定义

GET /wp/v2/taxonomies/<taxonomy>

示例请求

$ curl http://demo.wp-api.org/wp-json/wp/v2/taxonomies/<taxonomy>