## Welcome to React-on-Rails-API Guide

### 13.6.2017 

TL;DR

[Github rails-json-api](https://github.com/surajneupane55/rails-json-api-jwt-auth)

[Github React-frontend](https://github.com/surajneupane55/react-app-challenge-frontend)


Building a complete application with API is always a challenging job since every element must connect and work as intended. This guide will take you through the experience I encountered during building a simple React on Rails API. 

We will first build a small RESTful JSON Rails api that serves data which later will be consumed by our React Application.


### Building Rails application to consume data from React  

Rails 5 have evolved a lot where we can now have an option to creat api only. This option removes all the Middleware which are not necessary unlike native rails application. For example: Views are never created when we choose to create api application.

Our application will have resource name Record with following attributes: name, email and phone.

```markdown
Create Rails application

`$ rails new my_api --api`


Create Resource for Record 

`$ rails g resource Record name:string email:string phone:integer`

Migration to database

`$ rails db:create`
`$ rails db:migrate`

```

Next we want to set our routes in ```config/routes``` 


``` 

Rails.application.routes.draw do
  resources :records
end

```

If you notice above we don't define any root_path and that makes sense when we want to make our api stand-alone, stateless or sessionless which is holy grail of api functionality.

The other thing we are interested is to serialize our Record model. We want to make our json more readable, understandable and uniform which is taken care by

``` gem 'active_model_serializers', '~> 0.10.0' ```

There is a great [documentation](http://www.rubydoc.info/gems/active_model_serializers) which can be followed to get more information.


### Lets take a time to talk about JWT(JSON Web Token)

In our Record model we have some private data of user like phone and email. It is our responsibility to secure our api so only the authenticated user can get access to the Record Resource. 

Not long ago securing an api was a big deal where the user need to send their credentials and the token was generated and stored in database and every request need to fetch the token from database. Even worst was when we shared our api-token and used later to verify the authentic user. There is nothing wrong using token but the token have a tricky nature like when to make them expired or not, should we fetch the token from database or even should we store it in database.

After a long debate Devise dropped it token authentication for Rails because of its security vulnerability. Although Devise is still the most secure authenication handler in Rails application with session. 

JWT is the most simple best way to communicate with server from our frontend. The user credentials are send to Rails application which generate JSON token those token are send back to frontend which store them in local storage. Now, with every CURD request we attach this token and our rails api decode the token and let us access the Record Resource. The token was never stored in database and when user logOut the local storage of browser is cleared making it required for new token for different session. 

JWT never violate the basic principle of API where it should be stateless and stand-alone. Stateless in a sense that no cookies or session was used in anyway to authenticate the user. Stand-alone in a way that RESTful CURD JSON request was served without touching the database to authenticate the request. 

Ohhh... before I continue how to implement JWT for our rails app let me tell you one important thing 

Since our api is independent our rails app expect a CORS handling for making cross-origin AJAX possible so don't forget to uncomment ``` gem 'rack-cors' ```






















You can use the [editor on GitHub](https://github.com/surajneupane55/React-on-Rails-API-guide/edit/master/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)


For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/surajneupane55/React-on-Rails-API-guide/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
