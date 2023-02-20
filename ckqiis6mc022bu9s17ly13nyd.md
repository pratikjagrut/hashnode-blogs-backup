# Myreads, a digital bookshelf using Golang, React with Typescript, HarperDB, JWT based auth, and Docker

Hello everyone, to participate in HarperDB+Hashnode Hackathon I'm submitting this complete web-based application called Myreads which lets us maintain our offline bookshelf digitally. This is a full-stack application having backend, frontend, database service deployed using docker-compose.

# Little bit about me

My name is Pratik. I'm a system-level software developer and tester who works mainly with Linux and containers. I'm a Linux enthusiast and love open-source technologies. I don't have any professional experience in developing web-based applications. This project can be considered my first web-based project.

# Inspiration behind this project

For a long time, I was thinking to get started in the web development field as I already work on system-level programming with containers, adding web development to my portfolio could make it a full-stack developer. I also like to read books. Everyone who loves reading books keeps count of their books or maintains the status of books present on their shelf. So I thought why not build something in this area and I built the Myreads. Inspired from Goodreads

#  What is Myreads

Myreads is a simple but utilitarian web-based application that helps you to maintain your book status in three categories:

```
1. Wish List: Books which you want to read.
2. Reading List: Books you're currently reading.
3. Finished List: Books you finished reading.
```

# How I build it?

## Tech stack

Main tech stack:
1.  HarperDB as database service
2. Golang for backend server
3. React with Typescript for UI
4. Docker compose to deploy the application

Underlined tech stack:
1. git, git-submodules
2. go-fiber
3. harperdb-sdk-go
4. containers
5. react-bootstrap, react-query, etc...
6. HarperDB docker instance

## Challenges

As I'm new neophyte in web development, I faced lots of challenges.

- Learning frontend technologies

I come from more of a back-end technology background, so learning front-end and creating a project using it in a small period of time was a bit challenging. But after many sleepless nights and countless hours of going through tutorials, I got hang of it.

- HarperDB's support for Golang

When I started researching HarperDB, I found almost no tutorial about `Golang+HarperDB`. `This was one of my motivations to write backend using Golang so I could create some content of Golag+HarperDB`. I found one small library `harperdb-sdk-go`, which supported the basic CRUD functionality but does not provide all the functionalities needed to write the webserver which deals with the database. But I still managed somehow and wrote my own wrapper whenever it was needed.

- Model migration

Almost all the MVC based library supports the migrating model to the database table which was not yet supported by the harperdb-sdk-go. So I had to find another way to initialize the database using curl commands in a shell script.

## Myreads System Design

System design is simple and elegant. 

We've three services running: frontend, backend, and database.
Frontend makes the API call to the backend then backend verifies the authenticity of the API call and takes the required action accordingly.
 
### Myreads backend service

Myreads backend is written in `Golang` using `Fiber` and `harperdb-sdk-go`. Backend exposes a list of API which are then consumed by the frontend.

```
1. /api/register: For user registration.
2. /api/login: For use login.
3. /api/user: To fetch the logged-in user information.
4. /api/logout: To log out the user.
5. /api/books/add: To add book the database.
6. /api/books/all: To fetch all the books.
7. /api/books/reading: To fetch books from the reading list.
8. /api/books/wishlist: To fetch books from wishlist.
9. /api/books/finished: To fetch books from the finished list.
10. /api/books/deletebook: To delete book
11. /api/books/updatestatus: To update the status of the books such as reading->finished.
12. /api/static: To fetch the static content such as the image.
```

This is a standalone service so it can be spun up using docker and could be tested using POSTMAN or CURL.

### Myreads frontend service 

Myreads frontend is written using `React with Typescript`.
It consumes the APIs exposed by the backend.

This service is also standalone it can be used with any other backend service which will expose the same set of APIs as the Myreads backend service.

### Myreads database service 

Myreads backend service is hardcoded to use the HarperDB. The schema has two tables users and books. Users table is to store the user-related information such as name, email, and password. Books table is to store books related data such as name, author, description, imageLoc, userid of the book's owner.

# Application walkthrough

- Every new user will have to register themself at the registration page.

![Screenshot from 2021-06-26 02-52-37.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624995507176/bdZegExqg.png)

- After successful registration user will be redirected to the login page.

![Screenshot from 2021-06-26 02-52-34.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624995629977/UdVItDDyp.png)

- After login, the user will be redirected to the home page where he can see all the books from his bookshelf. 
![Screenshot from 2021-06-30 02-03-16.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624998814977/a-DZgAQ4E.png)

The new user will see the warning in the Red saying `empty bookshelf`. 
![Screenshot from 2021-06-30 02-04-40.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624998918476/hMSzqrmjd.png)

- User will need to fill a form to add books to the bookshelf.

![Screenshot from 2021-06-26 02-52-30.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624995862063/x1QA_Jylm.png)

- Then a user can visit the home page or the category page to see their book.
 Below is a screenshot of the `reading` category of the book.

![Screenshot from 2021-06-30 01-43-17.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624997669222/Ca5Ga2Vd-.png)

- Each book on the bookshelf has a `green colored more` button. On clicking it will prompt with the book description and two operation choices to perform on the book.
    
    1. Update the status of the book(wishlist, reading, finished)

    2. Remove the book from the bookshelf.

![Screenshot from 2021-06-30 01-58-33.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624998550105/2kx_W4jWN.png)

# Deploying the Myreads

This is a multiple service-based application so it makes sense to use containerization to deploy it. 

I've containerized frontend and backend services whereas the database service is handle by the HarperDB cloud instance. 

Follow the following steps to deploy the application.

1. Get the powerful free HarperDB instance by visiting [this link](https://studio.harperdb.io/sign-up)

2. Create New HarperDB Cloud Instance (one instance is free) and remember the `username` and `password` of the instance.

3. Once the instance is up and running click on the instance card and go to the `config` section.
    - In the config section, you'll see the `Instance URL` and `Instance API Auth Header 
       (this user)` copy them. They're needed to interact with DB.

4. Make sure you've [docker](https://docs.docker.com/engine/install/) and [docker-compose](https://docs.docker.com/compose/install/) installed and running successfully.

5. Clone the project repository from Github. This repo contains sub-modules of frontend and backend services so we'll need to clone it with `--recurse-submodules` flag. **Project source code: **  [https://github.com/pratikjagrut/myreads-harperdb](https://github.com/pratikjagrut/myreads-harperdb) 
```
git clone --recurse-submodules https://github.com/pratikjagrut/myreads-harperdb.git
cd myreads-harperdb
```
6. Update the `.env` file with your harperDB database credentials.
In the root of the `myreads-harperdb` directory, it has a `.env` file that contains the database and API connection-related environment variables. 
```
DB_HOST=
BASIC_AUTH_TOKEN=
HDB_ADMIN=
PASSWORD=
PORT=8000
API_URL=http://localhost:8000
```
`DB_HOST, BASIC_AUTH_TOKEN, HDB_ADMIN and PASSWORD` we've already obtained this information in the 2nd and 3rd steps.
```
DB_HOST=Instance URL from 3rd step
BASIC_AUTH_TOKEN=Instance API Auth Header (this user) from 3rd step
HDB_ADMIN=username from 2nd step
PASSWORD=password from 2nd step
```

7. If all the above steps were successfully followed then hit the docker-compose
```
docker-compose up
```

8. Once all the services are running perfectly fine you can visit the [http://locahost:3000/](http://locahost:3000/) and see the application is up and running.

![Screenshot from 2021-06-26 02-52-34.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1624995371201/w19mwn4ag.png)

*NOTE: If something goes wrong and you wish to check the logs of individual services run the following command.*
```
docker-compose logs server # To check the backed service logs
docker-compose logs web-ui # To check the frontend service logs
```