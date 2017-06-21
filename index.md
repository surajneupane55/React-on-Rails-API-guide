# Welcome to React-on-Rails-API Guide

### 13.6.2017 

TL;DR

[Github rails-json-api](https://github.com/surajneupane55/rails-json-api-jwt-auth)

[Github React-frontend](https://github.com/surajneupane55/react-app-challenge-frontend)

[DEMO](http://afraid-bedroom.surge.sh/)


Building a complete application with API is always a challenging job since every element must connect and work as intended. This guide will walk you through the experience I encountered during building a simple React on Rails API. 

We will first build a small RESTful JSON Rails API that serves data which later will be consumed by our React Application.


## Building Rails application to consume data from React  

Rails 5 has evolved a lot where we can now have an option to create API only. This option removes all the middleware which is not necessary unlike native rails application. For example: Views are never created when we choose to create API application.

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

If you noticed above we didn't define any root_path and that makes sense when we want to make our API stand-alone, stateless or sessionless which is holy grail of API functionality.

The other thing we are interested in is to serialize our Record model. We want to make our JSON more readable, understandable and uniform which is taken care by

``` gem 'active_model_serializers', '~> 0.10.0' ```

There is a great [documentation](http://www.rubydoc.info/gems/active_model_serializers) which can be followed to get more information.


### Lets take a time to talk about JWT(JSON Web Token)

In our Record model we have some private data of user, such as phone and email. It is our responsibility to secure our API, so only the authenticated user can get access to the Record Resource. 

Not long ago, securing an API was a big deal where the user needed to send their credentials and the token was generated and stored in database and every request needed to fetch the token from database. Even worst, we shared our API-Token which was later used to verify the authentic user. There is nothing wrong in using token but the token has a tricky nature like when to make them expired or not, should we fetch the token from database or should we even store it in database.

After a long debate Devise dropped token authentication for Rails because of its security vulnerability. Although, Devise is still the most secure authenication handler in Rails application with session. 

JWT is simple and the best way to communicate with server from our frontend. The user credentials are sent to Rails application which generates JSON token. Those tokens are sent back to frontend which stores them in local storage. Now, with every CURD request, we attach this token and our rails API decodes the token and lets us access the Record Resource. The token was never stored in database and when user logged out the local storage of browser is cleared making it necessary for new token for different user. 

JWT never violates the basic principle of API, where it should be stateless and stand-alone. Stateless in a sense that no cookies or session was used in anyway to authenticate the user and stand-alone, in a way that RESTful CURD JSON request was served without touching the database to authenticate the request. 

Ohhh... before I continue about how to implement JWT for our rails App, let me tell you one important thing: 

Since our API is independent our rails App expects a CORS handling for making cross-origin AJAX possible. So, don't forget to uncomment and bundle ``` gem 'rack-cors' ```

Implementing JWT in Rails is done with ```gem 'Knock' ```. Here is the [documentation](https://github.com/nsarno/knock) after implementing this. Now, we have the USER Resource added to our database. The routes have now changed and look like this: 

```
Rails.application.routes.draw do
  resources :records
  resources :users, only: [:create]
  post 'user_token' => 'user_token#create'
  mount Knock::Engine => "/knock"
end

```

Since any user data is handled by user_token#create Controller, we can now simply use a helper method to authenicate our Record resource:

```

class RecordsController < ApplicationController
  before_action :authenticate_user

  # GET /records
  def index
    @records = Record.all

    render json: @records
  end
  
```

### Pushing the API to HEROKU cloud

The easiest way to test our API is to deploy it to the cloud platform. I am using Heroku Cloud. It is fairly simple to implement with the guide from [Heroku](https://devcenter.heroku.com/articles/getting-started-with-rails5)

Here is the API of this [demo API application](https://arcane-oasis-17502.herokuapp.com/) which 
sounds crazy but it says page can't be found. But, it makes sense we have not implemented any view so the API can't serve us any view.

But, let's create a new user in  ``` heroku console ```

```
User.create(email:'abc@123.com',password:'securepassword')
D, [2017-06-14T09:59:36.139744 #4] DEBUG -- :    (0.8ms)  BEGIN
D, [2017-06-14T09:59:36.146241 #4] DEBUG -- :   SQL (4.6ms)  INSERT INTO "users" ("email", "password_digest", "created_at", "updated_at") VALUES ($1, $2, $3, $4) RETURNING "id"  [["email", "abc@123.com"], ["password_digest", "$2a$10$vbQPMjTWbclF6GKVR1.mEekxYTKePEz/tpFWKwcBjT6O/F/w4kfhG"], ["created_at", "2017-06-14 09:59:36.140222"], ["updated_at", "2017-06-14 09:59:36.140222"]]
D, [2017-06-14T09:59:36.148302 #4] DEBUG -- :    (1.4ms)  COMMIT
=> #<User id: 2, email: "abc@123.com", password_digest: "$2a$10$vbQPMjTWbclF6GKVR1.mEekxYTKePEz/tpFWKwcBjT6...", created_at: "2017-06-14 09:59:36", updated_at: "2017-06-14 09:59:36">
```
We have successfully created user, now we have to get the Api-token to access the Record so let's login as the user above:

```
surajs-MacBook-Pro:react-app-challenge-api surajnew55$ curl -H "Content-Type: application/json" -X POST -d '{"auth":{"email":"abc@123.com","password":"securepassword"}}' https://boiling-scrubland-97450.herokuapp.com/user_token
{"jwt":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0OTc1MjIxNjksInN1YiI6Mn0.aI9tvpyS6DtyAC_yY-pjbN9_mKXSNXoksOc9aYXAcho"}

```

Finally we got our JWT token back 
```
{"jwt":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0OTc1MjI2MTksInN1YiI6Mn0.WdjVw3InB9TbtpppymQ9C_3WypBC1oeaSd5sp7d2CdQ"}

```

We are now able to authenticate our CRUD request on Record Resource because we have valid JWT. So, let's do that:

```
surajs-MacBook-Pro:react-app-challenge-api surajnew55$ curl -i https://boiling-scrubland-97450.herokuapp.com/records  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0OTc1MjI2MTksInN1YiI6Mn0.WdjVw3InB9TbtpppymQ9C_3WypBC1oeaSd5sp7d2CdQ"
HTTP/1.1 200 OK
Server: Cowboy
Date: Wed, 14 Jun 2017 10:31:25 GMT
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Etag: W/"7857402c0029e1f4e397c717205f8a7b"
Cache-Control: max-age=0, private, must-revalidate
X-Request-Id: 05e9bce8-f3b2-4306-9920-ae57d7c37f75
X-Runtime: 0.031173
Vary: Origin
Transfer-Encoding: chunked
Via: 1.1 vegur

[{"id":1,"username":"abc123","email":"abc@123.com","phone":12345}]surajs-MacBook-Pro:react-app-challenge-api surajnew55$

```

Wow..... that was great. We got a list of Record with the use of token of JWT. 

Also, we must get 401 Unauthorized with wrong JWT 

```
surajs-MacBook-Pro:react-app-challenge-api surajnew55$ curl -i https://boiling-scrubland-97450.herokuapp.com/records  -H "Authorization: BearereyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0OTc1MjIxNjksInN1YiI6Mn0.aI9tvpyS6DtyAC_yY-pjbN9_mKXSNXoksOc9aYXAcho"
HTTP/1.1 401 Unauthorized

```

Finally we have successfully built an JSON API deployed it to Heroku cloud and it is ready to serve our React application.

# The new world of React

When I started this project I was totally new to React. I didn't know much about the core concept. I was confused either to use some architect like flux or redux. The truth is, I was overwhelmed with lots of information. But, eventually when all this information settled, React was one fun concept that circle around ```props``` and ```state```. Although, I have made a similar [CRUD single page todo application](https://radiant-coast-44956.herokuapp.com/) on angularjs but the experience was much fun and easy with React technology.

As a beginner, we should follow [React official documentation](https://facebook.github.io/react/) and video tutorials. The starting point is to clone the ```create-react-app```. 

The concept of Component and virtual DOM makes the React stand alone from other javascript library. Component is the building blocks of React app where each feature is segregated from one another.

In this application we are going to CURD the Record resource from our Rails API. But as we know Record is secured resource we must ask for valid JWT from our React app. So, our application will have two parent component ```Layout``` and ```Protected```. Layout component will handle all the views before the app is authenticated. While, Protected component will manage out Record resource along with CURD request on the API. 

This is how our ```Layout```looks like so far:

```
import React from 'react';

import logo from '../logo.svg';

import '../App.css';
import '../Skeleton.css';
import '../index.css';


import Body from './Body';
import Footer from './Footer'
import Header from './Header';


export default class Layout extends React.Component {


    render() {
        return (
            <div className="App">
                <div className="App-header">
                    <img src={logo} className="App-logo" alt="logo"/>
                    <h2>Welcome to React on Rails API</h2>
                </div>
                <Body/>
                <Header/><br/><br/>
                <Footer/>
            </div>
        );
    }
}


```

It is parent to static component view like ```Body```, ```Footer``` and ```Header``` with ```Links```

Let's take a moment to look how ```Header``` component is organized:

```import React from 'react';
import { Link } from 'react-router-dom'

export default class Header extends React.Component {
    render() {
        return (

            <header>
                <br/>
                <div className="container">
                    <div className="row">
                        <Link to='login' className="btn button-primary"><strong><h5>SignIn</h5></strong></Link>
                        {/*<Link to='signup'  className="btn  six columns" disabled>SignUp</Link>*/}
                    </div>
                </div>
            </header>
        );
    }
}
```
#Evolution of React Router v4

React Router v4 was just realized when I started this project. Although, there was not much blog support written by public, however [react training](https://reacttraining.com/react-router/web/guides/philosophy) site is the best place to start. React Router now is imported from ```react-router-dom``` where ```history``` from ```createBrowserHistory``` is passed as props and can be accessed from another component. Also, all our routes are now stacked inside ```<Router> ``` irrespective of protected component or not. There is a new way to authenticate ```Route```. i.e ```<Redirect>```. Now lets look our ```index.js``` after implementing above React Router feature:
 
```
import React from 'react';
import ReactDOM from 'react-dom'
import {  BrowserRouter as Router, Route, Redirect } from 'react-router-dom'
import {createBrowserHistory} from 'history';
import Layout from './components/Layout';
import Login from './components/Login';
// import Signup from './components/Signup';
import registerServiceWorker from './registerServiceWorker';
import ProtectedApp from './components/Protected'
import LoginActions from './components/LoginActions';



const app = document.getElementById('root');

LoginActions.loggedIn();

ReactDOM.render(
    <Router history={createBrowserHistory}>
        <div>
                <Route exact path="/" component={Layout} />
                <Route path="/login" component={Login} />
                {/*<Route path="/signup" component={Signup} />*/}
                <Route render={() => (
                this.loggedIn ? (
                    <Redirect to="/protected" component={ProtectedApp}/>
                ) : (
                        <Redirect to="/" component={Layout}/>
                    )
            )}/>
            <Route path="/protected" component={ProtectedApp} />
        </div>
    </Router>,
app);

registerServiceWorker();

``` 
We can see above how we have defined ```<Redirect>``` inside ```<Route>``` and checked with ```loggedIn``` method to check if user is authorized or not. 

```loggedIn```method is defined inside ```LoginAction``` component. Its main task is to make sure the user has valid JWT token stored in local storage. The ```loggedIn``` method looks like this:

```
export default {

    loggedIn: () => {
        let jwt = localStorage.getItem('jwt');
        if (jwt !== null) {
            return true;
        }
        return false;
    }
} 

```

By now we have successfully managed to get inside the ```Protected``` component. Protected component is the parent for ```CreateRecord```, ```ShowRecord``` and ```RecordItem```. RecordItem component handles each record and return the list of Record with the following code: 

```
return _.map(this.props.records, (record) => <RecordListItem key={record.id} {...record}
updateRecord={this.props.updateRecord} deleteRecord={this.props.deleteRecord}/>); 
  
```
  Above we have passed all the props that handle a single record from our ```Protected``` component, mapped each Record and return as  ```renderRecord ```. 
  
  Logic for sorting Record list is managed by ```ShowRecord``` component and replacing ```this.props.records``` with sorting list. The codes look as following:
  
  ``` sortByName() {

        var itemsList = [];
        _.map(this.props.records, (record) => itemsList.push(record));
        itemsList.sort(function (a, b) {
         
         //join first+last name and apply toLowerCase and compare
         
      return (a.username.replace(/\s+/g, "").toLowerCase() <b.username.replace(/\s+/g, "").toLowerCase()) ? -1 :                     (a.username.replace(/\s+/g, "").toLowerCase() > b.username.replace(/\s+/g, "").toLowerCase()) ? 1 : 0;

        });
        return itemsList;

    }

    sortedList() {
        this.setState({isSorted: true})
    } ```
    
    We have managed state ```isSorted``` to pass proper ```props``` that handels the sorted Record list.  
 
 Finally, lets take a look at how we will AJAX the list of record with ```axios``` in our Rails API.
 
 ```
     backendCall() {
        var config = {
            headers: {'Authorization': "Bearer " + localStorage.getItem('jwt')}
        };
        
        axios.get('https://arcane-oasis-17502.herokuapp.com/records', 
        config
        )
            .then((response) => {
            this.updateState(response);
            })
            .catch((error) => {
                console.log(error);
            })

    }
    updateState(response){
        this.setState({
            listRecord: response.data
        });

    }
            
 ``` 
 In the above code we made a function name ``` backendCall()```. This method sends get request to our base url with ```/records``` but ```axios``` takes config as second argument for get request so we build config as : 
 
 ```headers: {'Authorization': "Bearer " + localStorage.getItem('jwt')}```. 
 
 Here we fetched jwt from local storage and attached with Bearer for authorization in header of our request.
 
 The promise ```.then``` handles the response or error and finally response data is updated to the ```listRecord``` with ```setState```.
 
 Almost, the same process is repeated for ```Post```, ```Update``` and ```Delete``` Record. Axios have great [guide to follow](https://github.com/mzabriskie/axios) for these AJAX request.
 
 Finally our tour to React on Rails API is completed. Here is the [DEMO](http://afraid-bedroom.surge.sh/) App as final product of this tutorial. 
 
 
 ## 22.6.2017
 - Suraj Neupane
                                                      
                                                           
