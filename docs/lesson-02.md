建立 Entities 以及 CRUD
============================

1) 設定資料庫連線參數
-------------------

編輯 app/config/parameters.yml

```yml
parameters:
    database_driver: pdo_mysql
    database_host: 127.0.0.1
    database_port: null
    database_name: symfony
    database_user: root
    database_password: symfonyworkshop
```

設定好相關的連線參數，例如資料庫的帳號密碼以及資料庫名稱。

2) 建立資料庫
-------------

如果已經建立好資料庫，可以跳過此段。
透過下面的指令，Symfony 會根據你的連線設定建立好資料庫。

    app/console doctrine:database:create

3) 建立 Entity
-------------

在 Doctrine 中 Entity 就是對應資料庫中的 Table。
首先建立 Category Entity。

    app/console doctrine:generate:entity

接著 Symfony 會開始進行一連串的問答

```
First, you need to give the entity name you want to generate.
You must use the shortcut notation like AcmeBlogBundle:Post.

The Entity shortcut name: WorkshopBackendBundle:Category

Determine the format to use for the mapping information.

Configuration format (yml, xml, php, or annotation) [annotation]:
```

The Entity shortcut name 就填入 *WorkshopBackendBundle:Category*

註解方式就採用預設的 annotation

接著 Symfony 會開始詢問資料表的欄位

```
Available types: array, simple_array, json_array, object,
boolean, integer, smallint, bigint, string, text, datetime, datetimetz,
date, time, decimal, float, blob, guid.

New field name (press <return> to stop adding fields):
```

依序填入名稱以及欄位的資料類型，填完後直接按下 <return> (enter) 按鍵，結束對話。

這裡我們就只建立一個欄位 name 類型是 string 長度 255。

```
New field name (press <return> to stop adding fields): name
Field type [string]:
Field length [255]:
```

接著 Symfony 會確認一些設定，都採用預設值即可。
完成 Category Entity 的建立。

```
Do you want to generate an empty repository class [no]?


  Summary before generation


You are going to generate a "WorkshopBackendBundle:Category" Doctrine2 entity
using the "annotation" format.

Do you confirm generation [yes]?


  Entity generation


Generating the entity code: OK


  You can now start using the generated code!

```

再來同樣的方式建立 Post Entity

```
First, you need to give the entity name you want to generate.
You must use the shortcut notation like AcmeBlogBundle:Post.

The Entity shortcut name: WorkshopBackendBundle:Post

Determine the format to use for the mapping information.

Configuration format (yml, xml, php, or annotation) [annotation]:
```

接著我們要建立幾個欄位 subject, content, created_at, updated_at。
分別是文章的標題，內容，建立時間以及更新時間。

```
New field name (press <return> to stop adding fields): subject
Field type [string]:
Field length [255]:

New field name (press <return> to stop adding fields): content
Field type [string]: text

New field name (press <return> to stop adding fields): created_at
Field type [datetime]:

New field name (press <return> to stop adding fields): updated_at
Field type [datetime]:
```

同樣的完成最後建立步驟

```
Do you want to generate an empty repository class [no]?


  Summary before generation


You are going to generate a "WorkshopBackendBundle:Post" Doctrine2 entity
using the "annotation" format.

Do you confirm generation [yes]?


  Entity generation


Generating the entity code: OK


  You can now start using the generated code!


```

4) 建立資料表關聯
---------------

首先確立 Catgory 跟 Post 之間是一個一對多的關聯。也就是一個目錄底下會有很多篇的文章。

編輯 src/Workshop/Bundle/BackendBundle/Entity/Category.php

```php
<?php
namespace Workshop\Bundle\BackendBundle\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * Category
 *
 * @ORM\Table()
 */
class Category
{
    /**
     * @ORM\OneToMany(targetEntity="Post", mappedBy="category")
     * @var Post[]
     */
    protected $posts;
}
```

加入 Category 對 Post 的關聯 *OneToMany*

接著編輯 src/Workshop/Bundle/BackendBundle/Entity/Post.php

```php
<?php

namespace Workshop\Bundle\BackendBundle\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * Post
 *
 * @ORM\Table()
 * @ORM\Entity
 */
class Post
{

    /**
     * @ORM\ManyToOne(targetEntity="Category", inversedBy="posts")
     * @var Category
     */
    protected $category;
}
```

加入 Post 對 Category 的關聯 *ManyToOne*

注意 ManyToOne 關聯欄位設定名稱是 *inversedBy*。

5) 建立 Database Schema
----------------------

在實際寫入資料庫前，先驗證一下 Entity 的設定是否正確。

```
app/console doctrine:schema:validate

[Mapping]  OK - The mapping files are correct.
[Database] FAIL - The database schema is not in sync with the current mapping file.
The command terminated with an error status (2)
```

這時候 Symfony 回報，mapping 正確，可是資料庫還沒有同步。

就可以開始寫入資料庫了。

    app/console doctrine:schema:update --force

6) 建立 Entity 關聯存取 methods
------------------------------

建立完關聯後，我們還是缺乏存取的方式。
我們還得進行最後一個步驟，讓 Symfony 自動產生這些存取的 method。

```
app/console doctrine:generate:entities WorkshopBackendBundle:Category

Generating entity "Workshop\Bundle\BackendBundle\Entity\Category"
  > backing up Category.php to Category.php~
  > generating Workshop\Bundle\BackendBundle\Entity\Category
```

```
app/console doctrine:generate:entities WorkshopBackendBundle:Post

Generating entity "Workshop\Bundle\BackendBundle\Entity\Post"
  > backing up Post.php to Post.php~
  > generating Workshop\Bundle\BackendBundle\Entity\Post
```

 這時 Symfony 會重寫 Category.php 跟 Post.php，並且將原始檔案備份。

7) 建立 CRUD
------------------------------

接著要來建立 CRUD 嘍。
首先建立 Category CRUD。

    app/console doctrine:generate:crud

進入對話模式，照著填寫就對了。

```

  Welcome to the Doctrine2 CRUD generator



This command helps you generate CRUD controllers and templates.

First, you need to give the entity for which you want to generate a CRUD.
You can give an entity that does not exist yet and the wizard will help
you defining it.

You must use the shortcut notation like AcmeBlogBundle:Post.

The Entity shortcut name: WorkshopBackendBundle:Category

By default, the generator creates two actions: list and show.
You can also ask it to generate "write" actions: new, update, and delete.

Do you want to generate the "write" actions [no]? yes

Determine the format to use for the generated CRUD.

Configuration format (yml, xml, php, or annotation) [annotation]:

Determine the routes prefix (all the routes will be "mounted" under this
prefix: /prefix/, /prefix/new, ...).

Routes prefix [/category]:


  Summary before generation


You are going to generate a CRUD controller for "WorkshopBackendBundle:Category"
using the "annotation" format.

Do you confirm generation [yes]?


  CRUD generation


Generating the CRUD code: OK
Generating the Form code: OK


  You can now start using the generated code!


```


接著同樣的方式建立 Post CRUD。

```

  Welcome to the Doctrine2 CRUD generator



This command helps you generate CRUD controllers and templates.

First, you need to give the entity for which you want to generate a CRUD.
You can give an entity that does not exist yet and the wizard will help
you defining it.

You must use the shortcut notation like AcmeBlogBundle:Post.

The Entity shortcut name: WorkshopBackendBundle:Post

By default, the generator creates two actions: list and show.
You can also ask it to generate "write" actions: new, update, and delete.

Do you want to generate the "write" actions [no]? yes

Determine the format to use for the generated CRUD.

Configuration format (yml, xml, php, or annotation) [annotation]:

Determine the routes prefix (all the routes will be "mounted" under this
prefix: /prefix/, /prefix/new, ...).

Routes prefix [/post]:


  Summary before generation


You are going to generate a CRUD controller for "WorkshopBackendBundle:Post"
using the "annotation" format.

Do you confirm generation [yes]?


  CRUD generation


Generating the CRUD code: OK
Generating the Form code: OK


  You can now start using the generated code!


```

```
請注意，由於之前的 Default Controller 路徑會打架，記得修改一下。
只保留 indexAction，其餘的請拿掉。
```

src/Workshop/Bundle/BackendBundle/Controller/DefaultController.php

```php
<?php

namespace Workshop\Bundle\BackendBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Method;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

class DefaultController extends Controller
{
    /**
     * @Route("/", name="@BackendHome")
     * @Template()
     */
    public function indexAction()
    {
        return array();
    }

}
```

src/Workshop/Bundle/BackendBundle/Resources/views/Common/_header.html.twig

```jinja
<header role="banner" class="navbar navbar-inverse navbar-fixed-top bs-docs-nav">
    <div class="container col-md-10">
        <div class="navbar-header">
            <a class="navbar-brand" href="http://symfony.com/"><img style="height:18px;" src="{{asset('apple-touch-icon.png')}}" /></a>
        </div>

        <div class="navbar-header">
            <a class="navbar-brand" href="{{path('@BackendHome')}}">Home</a>
        </div>
        <nav role="navigation" class="collapse navbar-collapse bs-navbar-collapse">
            <ul class="nav navbar-nav">
            </ul>
        </nav>
    </div>
    <div class="container col-md-2">
        <nav role="navigation" class="collapse navbar-collapse bs-navbar-collapse">
            <ul class="nav navbar-nav pull-right">
                <li>
                    <a href="#">Logout xxx</a>
                </li>
            </ul>
        </nav>
    </div>
</header>
```

8) 客製化表單
------------

建立好的 CRUD 會位於 /admin/category, /admin/post。

可是選擇 /admin/post 的 *Create a new entry* 會發現當場噴出一堆 error。

這是因為 post 的欄位中有對應一個 Category Entity，然而 Category Entity 是一個物件，無法直接顯示*內容*。

解決方法就是幫 Category Entity 補上一個 __toString。

src/Workshop/Bundle/BackendBundle/Entity/Category.php

```php
<?php

namespace Workshop\Bundle\BackendBundle\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * Category
 *
 * @ORM\Table()
 * @ORM\Entity
 */
class Category
{
    public function __toString()
    {
        return $this->getName();
    }
}
```

接著我們要拿掉 Post 中 Created At 以及 Updated At 的欄位。

因為建立時間，跟更新時間是不能讓 User 自己修改才對。

src/Workshop/Bundle/BackendBundle/Form/PostType.php

```php
<?php

namespace Workshop\Bundle\BackendBundle\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolverInterface;

class PostType extends AbstractType
{
        /**
     * @param FormBuilderInterface $builder
     * @param array $options
     */
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('subject')
            ->add('content')
//            ->add('createdAt')
//            ->add('updatedAt')
            ->add('category')
        ;
    }
}
```

將建立以及更新時間拔除。

但是建立以及更新時間並不會自動維護，因此得動用 LifecycleCallbacks 功能。 詳細用法可以[參考官方說明][0]。

src/Workshop/Bundle/BackendBundle/Entity/Post.php

```php
<?php

namespace Workshop\Bundle\BackendBundle\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * Post
 *
 * @ORM\Table()
 * @ORM\Entity
 * @ORM\HasLifecycleCallbacks()
 */
class Post
{
    /**
     * @ORM\PrePersist
     */
    public function setCreatedAtValue()
    {
        $this->setCreatedAt(new \Datetime());
    }

    /**
     * @ORM\PrePersist
     * @ORM\PreUpdate
     */
    public function setUpdatedAtValue()
    {
        $this->setUpdatedAt(new \Datetime());
    }
}
```

9) 客製化樣板
------------

### Header Menu
src/Workshop/Bundle/BackendBundle/Resources/views/Common/_header.html.twig

```jinja
<header role="banner" class="navbar navbar-inverse navbar-fixed-top bs-docs-nav">
    <div class="container col-md-10">
        <div class="navbar-header">
            <a class="navbar-brand" href="http://symfony.com/"><img style="height:18px;" src="{{asset('apple-touch-icon.png')}}" /></a>
        </div>

        <div class="navbar-header">
            <a class="navbar-brand" href="{{path('@BackendHome')}}">Home</a>
        </div>
        <nav role="navigation" class="collapse navbar-collapse bs-navbar-collapse">
            <ul class="nav navbar-nav">
                <li><a href="{{path('category')}}">Category</a></li>
                <li><a href="{{path('post')}}">Post</a></li>
            </ul>
        </nav>
    </div>
    <div class="container col-md-2">
        <nav role="navigation" class="collapse navbar-collapse bs-navbar-collapse">
            <ul class="nav navbar-nav pull-right">
                <li>
                    <a href="#">Logout xxx</a>
                </li>
            </ul>
        </nav>
    </div>
</header>
```

### 套用 Layout

src/Workshop/Bundle/BackendBundle/Resources/views/Category/*.html.twig

src/Workshop/Bundle/BackendBundle/Resources/views/Post/*.html.twig

```jinja
{%extends "WorkshopBackendBundle:Layout:bootstrapLayout.html.twig"%}

{%block main%}
   {#...#}
{%endblock%}
```

### 表單元件

預設的表單元件樣板可能並不符合我們的需求，官方提供了簡單的方法讓我們可以自訂元件樣式。

src/Workshop/Bundle/BackendBundle/Resources/views/Layout/bootstrapLayout.html.twig

```jinja
{%extends "::base.html.twig"%}

{%if edit_form is defined%}
    {% form_theme edit_form 'WorkshopBackendBundle:Common:_form.html.twig' %}
{%endif%}
{%if delete_form is defined%}
    {% form_theme delete_form 'WorkshopBackendBundle:Common:_form.html.twig' %}
{%endif%}
{%if form is defined%}
    {% form_theme form 'WorkshopBackendBundle:Common:_form.html.twig' %}
{%endif%}

{%block javascripts%}
{{parent()}}
<script src="http://code.jquery.com/jquery-1.10.1.min.js"></script>
<script src="//netdna.bootstrapcdn.com/bootstrap/3.0.0/js/bootstrap.min.js"></script>
{%endblock%}

{%block stylesheets%}
{{parent()}}
<link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.0/css/bootstrap.min.css">
<link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.0/css/bootstrap-theme.min.css">
<link rel="stylesheet" href="{{asset('bundles/workshopbackend/css/main.css')}}">
{%endblock%}

{%block body%}
    {%block header%}{%include "WorkshopBackendBundle:Common:_header.html.twig"%}{%endblock%}
    <div class="container main-content">{%block main%}{%endblock%}</div>
    {%block footer%}{%include "WorkshopBackendBundle:Common:_footer.html.twig"%}{%endblock%}
{%endblock%}
```

透過 form_theme 引入自訂的表單元件樣板。接著我們得自訂一些表單樣板。

src/Workshop/Bundle/BackendBundle/Resources/views/Common/_form.html.twig

```jinja
{%block form%}
    {{ form_start(form, {attr: {role: 'form'}}) }}
        {{ form_widget(form) }}
    {{ form_end(form) }}
{%endblock form%}

{%block form_row%}
    <div class="form-group">
        {{ form_label(form) }}
        {{ form_errors(form) }}
        {{ form_widget(form, {attr: {class: 'form-control'}}) }}
    </div>
{%endblock form_row%}

{%block button_row%}
    <div class="form-group">
        {{ form_widget(form, {attr: {class: 'btn btn-default'}}) }}
    </div>
{%endblock%}
```

更多可用的表單樣板可以參考[這個檔案][1]

10) 分頁
--------

Doctrine 並沒有提供分頁器的功能，因此得透過第三方的 Bundle 來實現。

這邊我們選用了 [knplabs/knp-paginator-bundle][2] 這個 Bundle 來協助我們實現分頁功能。

```
composer.phar require knplabs/knp-paginator-bundle

Please provide a version constraint for the knplabs/knp-paginator-bundle requirement: 2.3.*
```

版本就使用 2.3.*

接著要啟用這個 Bundle

app/AppKernel.php

```php
<?php

use Symfony\Component\HttpKernel\Kernel;
use Symfony\Component\Config\Loader\LoaderInterface;

class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = array(
            //...
            new Knp\Bundle\PaginatorBundle\KnpPaginatorBundle(),
        );

```

接著修改查詢

src/Workshop/Bundle/BackendBundle/Controller/CategoryController.php

```php
<?php

/**
 * Category controller.
 *
 * @Route("/category")
 */
class CategoryController extends Controller
{

    /**
     * Lists all Category entities.
     *
     * @Route("/", name="category")
     * @Method("GET")
     * @Template()
     */
    public function indexAction()
    {
        $em = $this->getDoctrine()->getManager();
        try {
            $query = $em->getRepository('WorkshopBackendBundle:Category')
                    ->createQueryBuilder('c')
                    ->getQuery();

            $paginator = $this->get('knp_paginator');
            $pagination = $paginator->paginate(
                    $query, $this->getRequest()->get('page', 1), 10 //10 -> 每頁幾筆
            );
        } catch (QueryException $e) {
            throw $this->createNotFoundException();
        }
        return array(
            'pagination' => $pagination,
        );
    }

```

修改樣板加入 pagination

src/Workshop/Bundle/BackendBundle/Resources/views/Category/index.html.twig

```jinja
{%extends "WorkshopBackendBundle:Layout:bootstrapLayout.html.twig"%}

{% block main %}
<h1>Category list</h1>

<table class="records_list table table-striped">
    <thead>
        <tr>
            <th>{{ knp_pagination_sortable(pagination, 'Id', 'c.id') }}</th>
            <th>{{ knp_pagination_sortable(pagination, 'Name', 'c.name') }}</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        {% for entity in pagination %}
        <tr>
            <td><a href="{{ path('category_show', { 'id': entity.id }) }}">{{ entity.id }}</a></td>
            <td>{{ entity.name }}</td>
            <td>
                <ul class="list list-inline">
                    <li>
                        <a href="{{ path('category_show', { 'id': entity.id }) }}" class="btn btn-default"><i class="glyphicon glyphicon-eye-open"></i> Show</a>
                    </li>
                    <li>
                        <a href="{{ path('category_edit', { 'id': entity.id }) }}" class="btn btn-default"><i class="glyphicon glyphicon-edit"></i> Edit</a>
                    </li>
                </ul>
            </td>
        </tr>
        {% endfor %}
    </tbody>
</table>

{{ knp_pagination_render(pagination, "KnpPaginatorBundle:Pagination:twitter_bootstrap_v3_pagination.html.twig") }}

<ul class="list list-inline">
    <li>
        <a href="{{ path('category_new') }}" class="btn btn-default">
            <i class="glyphicon glyphicon-plus-sign"></i> Create a new entry
        </a>
    </li>
</ul>
{% endblock %}
```

[0]:    http://symfony.com/doc/current/book/doctrine.html#lifecycle-callbacks
[1]:    https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/form_div_layout.html.twig
[2]:    https://github.com/KnpLabs/KnpPaginatorBundle
