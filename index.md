## Welcome to React-on-Rails-API Guide

### 13.6.2017 

Building a complete application with API is always a challenging job since every element must connect and work as intended. This guide will take you through the experience I encountered during building a simple React on Rails API. 

We will first build a small RESTful JSON Rails api that serves data which later will be consumed by our React Application.


### Building Rails application to consume data from React  

Rails 5 have evolved a lot where we can now have an option to creat api only. This option removes all the Middleware which are not necessary unlike native rails application. For example: Views are never created when we choose to create api application.

Our application will have resource name Record with following attributes: name, email and phone.

```markdown
Create Rails application

`$ rails new my_api --api`


Create Resource for Record 

`$ rails g model Record name:string email:string phone:integer`





# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```


















 You can use the [editor on GitHub](https://github.com/surajneupane55/React-on-Rails-API-guide/edit/master/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
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
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/surajneupane55/React-on-Rails-API-guide/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
