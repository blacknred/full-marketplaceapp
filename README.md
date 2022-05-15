# Marketplace app

Microservice boilerplate for marketplace app

## Architecture

> npm monorepo(lerna, yarn workspaces), docker/cubernates

| Services              | Container            | Stack                           | Ports  |
| --------------------- | -------------------- | ------------------------------- | ------ |
| Redis                 | redis                | Redis                           | 6379   |
| Queue                 | rabbitmq             | RabbitMQ                        | 5672   |
| Postgres              | postgres             | Postgres                        | 5432   |
| MongoDB               | mongo                | MongoDB                         | 27017  |
| Static, proxy, cache  | nginx                | Nginx                           | 80/443 |
| **Packages**          |                      |                                 |        |
| App Type Definitions  | types                | TS                              | -      |
| Core microservice     | core-service         | TS, NestJS                      | -      |
| UI Design system      | ui                   | TS, React, Tailwind, Storybook  | -      |
| **Services**          |                      |                                 |        |
| User CRUD service     | user-service         | TCP, Pg, core-service           | 8081   |
| Auth CRUD service     | auth-service         | TCP, Redis, core-service        | 8082   |
| Shop CRUD service     | shop-service         | TCP, Pg, core-service           | 8083   |
| Product CRUD service  | product-service      | TCP, Mongo, core-service        | 8084   |
| Order CRUD service    | order-service        | AMQP, Mongo, core-service       | 8085   |
| Notification service  | notification-service | AMQP, core-service              | 8086   |
| API Gateway, Api docs | gateway              | Http, REST, Swagger, TS, NestJS | 8080   |
| Web client            | web                  | TS, NextJS(ssr), Swr, ui/web    | 3000   |
| Docs web client       | web-docs             | TS, NextJS(isr), ui/web         | 3001   |
| Shop web client       | web-shop             | TS, Webpack(csr), Redux, ui/web | 3002   |
| Admin web client      | web-admin            | TS, Webpack(csr), Redux, ui/web | 3003   |

## Packages

### App Type Definitions

### Core microservice

### UI Design system

- tokens
  - sizes[]
  - colors[]
- icons
- hooks[]
- components[Button,ActionButton,Table,List,Card,Selector,Multiselector,PaginationBlock,Tabs,Form,Input]

## Services

### BE

- **Api Gateway**
  - STACK: http transport, user-service, auth-service, shop-service, product-service, order-service
  - API
    - feed
      - GET /api/v1/products?promoted=true&limit=20&offset=2&'sort.field'=createdAt&'sort.order'=DESC
      - GET /api/v1/shops?promoted=true&limit=20&offset=2&'sort.field'=createdAt&'sort.order'=DESC
      - GET /api/v1/highlights?promoted=true&limit=20&offset=2&'sort.field'=createdAt&'sort.order'=DESC
      - GET /api/v1/brands?promoted=true&limit=20&offset=2&'sort.field'=createdAt&'sort.order'=DESC
    - search: products, barnds, shops, highlights
    - auth
    - users
    - shops
    - brands
    - highlights
    - products
    - orders
- **User CRUD microservice**
  - STACK: tcp transport, pg
  - MODELS
    - `User`[id!,firstname!,secondname!,email!,confirmed!,password,phone?,locationId?,address,avatar?,2fa?,deletedAt?]
    - `Notification`[]
  - API
    - create
    - read
    - update
    - delete: soft delete
- **Auth CRUD microservice**
  - STACK: tcp transport, redis
  - MODELS: `Session`[]
  - API
    - create: login
    - read: user info
    - delete: logout
- **Shop CRUD microservice**

  - STACK: tcp transport, pg
  - MODELS

    - `Shop`[id,slug!,name!,description?,rating(1-5)!,createdAt!,image!,promoted!,deletedAt?,locationId!,partnership(s|sd|sdd)!]
    - `Employee`[id!,userId!,shopId!,role!]
    - `Depot`[id!,name!,locationId!,geo!,decription?,image?,createdAt!,deletedAt?]
    - `PuPoint`[id!,name!,locationId!,geo!,decription?,image?,createdAt!,deletedAt?,storeDays!]

    - `Location`[id!,name!,createdAt!,geo!]

    - `Coupon`[id!,code!,shopId!,discount!,validPrice?,endsAt!,count!]
    - `UserCoupon`[id!,userId!,couponId!]
    - `Brand`[id!,slug!,name!,image!,promoted!]
    - `Highlight`[id!,slug!,name!,image!,endsAt?,promoted!]

  - API:
    - create
      - cooperation types:
        - S (showcase): we act as a trading platform, and storage, assembly and delivery of orders are on your side
        - SD (showcase & delivery): store goods in your warehouse, collect orders and transfer finished packages to Ozon for delivery to customers.
        - SDD (showcase & depot & delivery): storage, assembly and delivery of goods - on our side. All you have to do is send us the goods.
      - role model:
        - agent: depot worker or courier, access to orders in some statusses
        - manager: agent + [orders,products,questions,reviews,highlights]
        - admin: manager + [employes,coupons,analitics,finance,shop]
    - read
    - update
    - delete: soft delete

- **Product CRUD microservice**

  - STACK: tcp transport, mongo(embedding)
  - MODELS

    - `Product`[id!,slug!,shopId!,brandId!,categoryId!,name!,description(md)!,count!,rating(0-5)!,views(0),createdAt!,deliveryDays!,promoted!,deletedAt?,medias[url?],prices[{price!,discount?,createdAt!}],highlights[highlightId?],features[{feature,value}]]
    - `Feature`[id!,label!,value!,productId!]
    - `Category`[id!,slug!,categoryId,name!,description?,image?,createdAt!,features!{}]

    - `Discussion`[id!,userId!,productId!,hideUser?,likes(0),createdAt!,text!,responses[Discussion?]]
    - `Review`[id!,userId!,productId!,hideUser?,likes(0),createdAt!,responses[Discussion?],pros!,cons!,comment(md)?,rating(1-5)!]
    - `UserProduct`[id!,userId!,productId!]
    - `CardProduct`[id!,userId!,productId!,count!]

  - API

    - create
    - read
      /api/v1/pro

      - content: [categoriesTree, priceRange, brandsMultiselect, shopsMultiselect, highlightsMultiselect, ...categoryFeaturesMultiselect]
      - req:
        - filters: text=grass|category=3242|brand=8978789|shop=869879|highlight=23489
      - res:
        {
        status: 200,
        errors: null,
        data: {
        categories: [],
        minPrice: 3088
        maxPrice: 7088
        brands: [],
        shops: [],
        highlights: [],
        [color,producer,size,material...]
        }
        }

      api/v1/products?

      - content: [sorting, product grid/table, pagination]
      - req:
        - sorting: 'sort.field'=createdAt|rating|views|price&'sort.order'=DESC|ASC
        - pagination: limit=20&offset=2
        - filters:
          text=grass
          category=3242
          brand=8978789,3453
          shop=869879,345
          highlight=23489,456465
      - res:
        {
        status: 200,
        errors: null,
        data: {
        items: []
        total: 290
        hasMore: true,
        }
        }

    - update
    - delete: soft delete

- **Order CRUD microservice**
  - STACK:
    - amqp tarnsport[soft load, batching(prefetchCount), schedule(rabbitmq_delayed_message_exchange)]
    - mongo(embedding)
    - notification-service
  - MODELS:
    - `Order`[id!,shop!{id,name},user!{id,name,avatar},count!,price!,discount?,total!,createdAt!,deliveryAt!,product{Product}!,pupoint?{PuPoint},statusses![{status!(created|canceled|collected|delivering|in_pupoint|delivered|lost|returning|closed|completed|issue|return|refund|refunded),user!{id,name,avatar},comment?,createdAt!}]]
  - API
    - create
    - read
    - update: only statusses;with notifications;available transitions:
      - _depot_
      - CREATED => COLLECTED(agent)|CANCELED(user, manager/admin:no_product, system:deliveryAt)
      - COLLECTED => DELIVERING(agent)|CANCELED(user, manager/admin:no_delivery, system:deliveryAt)
      - _delivering_
      - DELIVERING => IN_PUPOINT(agent)|DELIVERED(agent)|LOST(employee)|CANCELED(system:deliveryAt)
      - IN_PUPOINT => DELIVERED(agent)|RETURNING(agent:storeDays)|LOST(employee)
      - DELIVERED => COMPLETED(system:1d)|ISSUE(user:bid)
      - ISSUE => RETURN|REFUND|COMPLETED(system)
      - _returning_
      - RETURN => RETURNING(agent)|COMPLETED(system:3d)
      - RETURNING => RETURNED(agent)|LOST(employee)
      - RETURNED => REFUND|COMPLETED(system:1d:return?)
      - _refund_
      - CANCELED => REFUND(system:1d)
      - LOST => REFUND|COMPLETED(system:1d:return?)
      - REFUND => REFUNDED(system)
      - REFUNDED => COMPLETED|CLOSED(system:1d:delivered|in_pupoint)
      - _for completed orders ask review_
- **Notification microservice**
  - STACK:
    - amqp transport[soft load, batching(prefetchCount), acknowledge]
    - auth-service
    - user-service
  - API:
    - email: ack if email successfully sent
    - sms: ack if sms successfully sent
    - push: ack if push not available(auth-service) || successfully sent
    - notify:
      - ack if successfully sent notification in user-service
      - enqueue push notifications
      - enqueue email notifications

### FE

#### Web client (marketplaceapp, isr/ssr)

1. **Account layout**(form)
2. **Main layout**(header, content[NonProductSelection, ProductSelection, ProductDetails, Card, Order, My], footer)
   - `HEADER`
     - DATA: GET /api/v1/catalogs/categories, GET /api/v1/auth, GET card?
     - CONTENT: Marketplaceapp, categories, searchbar[in products, list shops/brands/highlights], profile[orders, notifications, profile, logout], favourites, cart
     - ACTIONS: search(GET /api/v1/search?text=grass), link(/categories/:id, /search?text=grass, /categories/:id?text=grass, /my/orders, /my/notifications, /my, /my/favourites, /my/cart), logout(DELETE /api/v1/auth)
   - `CONTENT`
     - `/`
       - DATA: GET /api/v1/feed?limit=20&offset=2
       - CONTENT: infinite grid
       - ACTIONS: link(/products/:id, /highlights/:id, /brands/:id, shops/:id), infiniteLoading(), addToCard()
     - `/categories`
       - DATA:
       - CONTENT:
       - ACTIONS:
     - `/search?text=grass`
       - DATA: GET /api/v1/products?text=grass&limit=20&offset=2&'sort.field'=createdAt&'sort.order'=DESC
       - CONTENT: [categories, filters], [sorting, grid, offset pagination]
       - ACTIONS: link(/products/:id, /category/:id), changeFilter(), changeSorting(), pagination()
     - `/categories/:id`
       - DATA: GET /api/v1/products?categoryId=${id}&limit=20&offset=2&'sort.field'=createdAt&'sort.order'=DESC&text=grass
       - CONTENT: [categories, filters], [sorting, grid, offset pagination]
       - ACTIONS: link(/products/:id, /category/:id), changeFilter(), changeSorting(), pagination()
     - `/brands`
       - DATA: GET /api/v1/brands?'sort.field'=name&'sort.order'=DESC
       - CONTENT: list by alphabet
       - ACTIONS: link(/brands/:id)
       - `/brands/:id`
         - DATA: GET /api/v1/products?brandId=${id}&limit=20&offset=2&'sort.field'=createdAt&'sort.order'=DESC
         - CONTENT: [categories, filters], [sorting, grid, offset pagination]
         - ACTIONS: link(/products/:id, /category/:id), changeFilter(), changeSorting(), pagination()
     - `/highlights`
       - DATA: GET /api/v1/highlights?limit=20&offset=2&'sort.field'=promoted&'sort.order'=DESC
       - CONTENT: grid, offset pagination
       - ACTIONS: link(/highlights/:id), pagination()
       - `/highlights/:id`
         - DATA: GET /api/v1/products?highlight=${id}&limit=20&offset=2&'sort.field'=createdAt&'sort.order'=DESC
         - CONTENT: [categories, filters], [sorting, grid, offset pagination]
         - ACTIONS: link(/products/:id, /category/:id), changeFilter(), changeSorting(), pagination()
     - `/shops`
       - DATA: GET /api/v1/shops?limit=20&offset=2&'sort.field'=promoted&'sort.order'=DESC
       - CONTENT: grid, offset pagination
       - ACTIONS: link(/shops/:id)
       - `/shops/:id`
         - DATA: GET /api/v1/products?shopId=${id}&limit=20&offset=2&'sort.field'=createdAt&'sort.order'=DESC
         - CONTENT: [categories, filters], [sorting, grid, offset pagination]
         - ACTIONS: link(/products/:id, /category/:id), changeFilter(), changeSorting(), pagination()
     - `/product/:id`
       - DATA: GET /api/v1/products/:id, GET /api/v1/
       - CONTENT: preview[medias, description, delivery_conditions], feautures, reviews/discussions(PATCH /api/v1/products/:id), previously_viewed/promoted?
       - ACTIONS:
     - `/card`
       - DATA: GET /api/v1/cardproducts
       - CONTENT: list, checkoutBlock
       - ACTIONS: deleteProduct(DELETE /api/v1/cardproducts/:id), changeProductCount(PATCH /api/v1/cardproducts/:id), selectProducts(), link(/checkout)
     - `/checkout?`
       - DATA:
       - CONTENT:
       - ACTIONS: payOrder(_strapi_, POST /api/v1/orders)
     - `/location`
       - DATA: GET /api/v1/products/:id, GET /api/v1/
       - CONTENT: preview[medias, description, delivery_conditions], feautures, reviews/discussions(PATCH /api/v1/products/:id), previously_viewed/promoted?
       - ACTIONS:
     - `/my`
       - DATA: -
       - CONTENT: link grid
       - ACTIONS: link(/my/...)
       - `/my/notifications` GET
       - `/my/orders`
         - DATA: GET /api/v1/orders
         - CONTENT: list
         - ACTIONS: link(/orders/:id)
         - `/my/orders/:id` GET api/v1/orders/:id
       - `/my/wishlist` GET /api/v1/userproducts
       - `/my/coupons` GET /api/v1/usercoupons
       - `/my/payments` GET /api/v1/users
       - `/my/reviews` GET /api/v1/reviews?userId={id}
       - `/my/discussions` GET /api/v1/discussions?userId={id}
       - `/my/profile` GET /api/v1/users
       - `/my/settings` GET /api/v1/users
   - `FOOTER`
     - DATA: -
     - CONTENT: become a seller, apps, lang, accessibility
     - ACTIONS: link(/seller.marketplaceapp), link(appstore,google play), changeLang(), changeAccessibility()

#### Shop web client (seller.marketplaceapp, csr)

1. **Account layout**(form)
   - `/account`(!A)
     - DATA: GET /api/v1/auth => redirect to /orders if allready logged
     - CONTENT: form[email,password]
     - ACTIONS: login(POST /api/v1/auth), link(/account/new), link(/acount/restore)
   - `/account/new`(!A)
     - DATA: GET /api/v1/auth => redirect to /orders if allready logged
     - CONTENT: form[token,name,password]accept the offer
     - ACTIONS: register(POST /api/v1/users), link(/account)
   - `/account/restore`(!A)
     - DATA: GET /api/v1/auth => redirect to /orders if allready logged
     - CONTENT: form[token,password,password2]
     - ACTIONS: restore(PATCH /api/v1/users), link(/account)
   - `/account/confirm`
     - DATA:
     - CONTENT: form[email]
     - ACTIONS:
   - `/account/edit`(A)
     - DATA:
     - CONTENT: form[firstname,secondname,address,phone?,country?,2fa]
     - ACTIONS:
   - `/account/change-email`(A)
     - DATA:
     - CONTENT: form[]
     - ACTIONS:
   - `/account/change-password`(A)
     - DATA:
     - CONTENT: form[]
     - ACTIONS:
2. **Main layout**(header, tabs)
   - `HEADER`
     - DATA: GET /api/v1/
     - CONTENT: Marketplaceapp Shopname:rating, user[settings,lang,logout]
     - ACTIONS: link(/account/edit), logout(DELETE /api/v1/auth), changeLang()
   - `CONTENT(TABS)`
     - `/orders`(A)
       - DATA: GET /api/v1/orders?sellerId=${id}&limit=20&offset=2&'sort.field'=createdAt&'sort.order'=DESC&query=first
       - CONTENT: table[,editbtn], filters, sorting, offset pagination
       - ACTIONS: link(/orders/:id, /orders/:id?edit=true), changeFilter(), changeSorting(), pagination()
       - `/orders/:id`(A)
         - DATA: GET /api/v1/orders/:id, catalogs
         - CONTENT: card[]
         - ACTIONS: edit(PATCH /api/v1/orders/:id)
     - `/products`(A)
       - DATA: GET /api/v1/products?sellerId=${id}&limit=20&offset=2&'sort.field'=createdAt&'sort.order'=DESC&query=first
       - CONTENT: table[,editbtn], filters, sorting, offset pagination, new product link
       - ACTIONS: link(/products/new, /products/:id, /products/:id?edit=true), changeFilter(), changeSorting(), pagination()
       - `/products/new`(A)
         - DATA: catalogs
         - CONTENT: card[]
         - ACTIONS: save(POST /api/v1/products)
       - `/products/:id`(A)
         - DATA: GET /api/v1/products/:id, GET /api/v1/reviews?productId=${id}, catalogs
         - CONTENT: card[]
         - ACTIONS: edit(PATCH /api/v1/products/:id), delete (DELETE /api/v1/products/:id)
     - `/questions`(A)
       - DATA:
       - CONTENT: table[]
       - ACTIONS:
     - `/reviews`(A)
       - DATA:
       - CONTENT: table[]
       - ACTIONS:
     - `/analitics`(A)
       - DATA: ?
       - CONTENT:
         - cards[], charts[]
         - orders per day/week/month/year/all category/brand/highlight
         - products per views/sales/rating/questions/reviews
       - ACTIONS: save(reports as pdf/xsl)
     - `/finance`(A)
       - DATA: ?
       - CONTENT: card[]
       - ACTIONS: ?
     - `/employees`(A)
       - DATA: ?
       - CONTENT: card[]
       - ACTIONS: ?
       <!-- - `/notifications`(A)
       - DATA: ?
       - CONTENT: card[]
       - ACTIONS: link(/${targetlink}) -->

#### Admin web client (admin.marketplaceapp, csr)

-moderation new product/discussion/review/brand/shop/highlight active(true|'')
-catalogs? categories management
-shops management
-promotion management
-order bids
-finance
-analitics

1. **Account page**(middle form)

2. **Main page**(header, tabs)
   - `/orders`(A)
     - DATA: GET /api/v1/orders?sellerId=${id}&limit=20&offset=2&'sort.field'=createdAt&'sort.order'=DESC&query=first
     - CONTENT: table[,editbtn], filters, sorting, offset pagination
     - ACTIONS: link(/api/v1/orders/:id), edit(/api/v1/orders/:id)
     - `/orders/:id`(A)
       - DATA: GET /api/v1/orders/:id, catalogs
       - CONTENT: card[]
       - ACTIONS: edit(PATCH /api/v1/orders/:id)
   - `/moderation`(A)
     - DATA: GET /api/v1/products?sellerId=${id}&limit=20&offset=2&'sort.field'=createdAt&'sort.order'=DESC&query=first
     - CONTENT: table[,editbtn], filters, sorting, offset pagination
     - ACTIONS: link(/api/v1/products/:id), editbtn(/api/v1/products/:id)
     - `/newproducts/:id`(A)
       - DATA: GET /api/v1/products/:id, GET /api/v1/reviews?productId=${id}, catalogs
       - CONTENT: card[]
       - ACTIONS: edit(PATCH /api/v1/products/:id), delete (DELETE /api/v1/products/:id)
   - `/catalogs`(A)
     - DATA:
     - CONTENT:
     - ACTIONS:
   - `/analitics`(A)
     - DATA: ?
     - CONTENT: cards[], charts[]
     - ACTIONS: save(reports as pdf/xsl)
   - `/finance`(A)
     - DATA: ?
     - CONTENT: card[]
     - ACTIONS: ?

#### Docs web client (docs.marketplaceapp, isr)

1. **Main layout**
   - `HEADER`
     - DATA: local page.html
     - CONTENT: Marketplaceapp Docs, searchbar, lang, accessibility
     - ACTIONS: search(), changeLang(), changeAccessibility()
   - `CONTENT`
     - `/`
       - DATA: [index].html
       - CONTENT: all pages links
       - ACTIONS: link(/:pagename)
     - `/:pagename`
       - DATA: [pagename].html
       - CONTENT: text & media
       - ACTIONS: -
   - `FOOTER`
     - DATA: check rating
     - CONTENT: Is it helpful btns
     - ACTIONS: incrementRating/decrementRating()

## Run the project

### Setup

1. Fork/Clone this repo

1. Download [Docker](https://docs.docker.com/docker-for-mac/install/) (if necessary)

### Build and Run the App

1. Set the Environment variables in .env.dev

1. Fire up the Containers

   ```sh
   make network
   make dev-check
   make dev
   ```

### Production

1. Set the Environment variables in .env

1. Run the containers:

   ```sh
   make network
   make prod-build
   make prod
   ```

@marketplaceapp/types
@marketplaceapp/types
@marketplace/ui

@marketplace/user-service
@marketplace/auth-service
@marketplace/shop-service
@marketplace/product-service
@marketplace/order-service
@marketplace/notification-service
@marketplace/gateway
@marketplace/web
@marketplace/docs-web
@marketplace/shop-web
@marketplace/admin-web
