# Transforming normal Github profile into a dynamic portfolio

Today Github is the behemoth platform for open-source projects. There would not be a single person in the open-source community who doesn't use GitHub. And there could be a minuscule number of developers who don't have a GitHub profile. GitHub is not only home for the best open-source projects but also a place where developers could showcase their talent via a panoply of projects.

Below is the default, pedestrian GitHub profile page that shows the name, contact information, and some repositories.

![GitHub default profile page](https://github.com/pratikjagrut/blog/raw/main/static/img/git/ghRegProf.png align="left")

A few months back, Github launched a new feature that lets us customize our profile page using the README file. README is a repository or project-level file used for documenting the project. README.md is written in `Markdown(lightweight markup language)` that converts text to HTML.

#### In this blog post, I will show how I customized my mundane Github profile using the README file.

[![GitHub default profile page](https://github.com/pratikjagrut/blog/raw/main/static/img/git/ghNewProf.png align="left")](https://github.com/pratikjagrut)

## Create a new repository with the username

```plaintext
1. Make sure your repository name matches the Github username.
2. Make a public repository.
3. Add README.md to it.
```

![Create repo](https://github.com/pratikjagrut/blog/raw/main/static/img/git/repoC.png align="left")

## Adding a banner image

When someone opens my profile, I wanted to greet them with a good banner. I created a banner on [***canva.com***](https://www.canva.com/). They have a good collection of free templates.

![Create repo](https://github.com/pratikjagrut/blog/raw/main/static/img/git/ghbanner.png align="left")

I'm keeping this image in the `img` directory. Now I want it to be at the top, so I put the link to the banner image on the first line of my README file.

To add an image in the README file, use the following syntax.

```plaintext
![Pratik's GitHub Banner](./img/banner1.png)
```

If you want to make an image, a link that will open your blog or some other sites, use the following syntax.

```plaintext
[![Pratik's GitHub Banner](./img/banner1.png)](https://pratikjagrut.dev)
```

## Add a bio

GitHub initializes README with a few bullet points. I used the same template to add my bio.

README file is highly customizable using Markdown. Checkout the [***Markdown syntax guide***](https://www.markdownguide.org/basic-syntax/)

![Bio](https://github.com/pratikjagrut/blog/raw/main/static/img/git/bio.png align="left")

I've added some badges under look me up bullet point, which I created using [***shields.io***](http://shields.io/). They lets us design customize badges with the links or without links. I used badges from [***social category***](https://shields.io/category/social).

Once we've got the badge, we can then put it anywhere we want.

```plaintext
[![Linkedin Badge](https://img.shields.io/badge/-pratikjagrut-0e76a8?style=flat&labelColor=0e76a8&logo=linkedin&logoColor=white)][linkedin]
```

## Listing latest blog posts

[***Gautam krishna R***](https://github.com/gautamkrishnar) created a Github action workflow which lets us show our latest blog, latest, youtube video, latest StackOverflow activity and many other things. [***Follow the documentation***](https://github.com/gautamkrishnar/blog-post-workflow).

![Blogs lists](https://github.com/pratikjagrut/blog/raw/main/static/img/git/blogs.png align="left")

## Showing GitHub statistics

[***Github Readme stats***](https://github.com/anuraghazra/github-readme-stats) shows the statistics of our profile in various beautiful cards.

![Github stats](https://github.com/pratikjagrut/blog/raw/main/static/img/git/stats.png align="left")

##### There are lots of customizations we can do with the GitHub profile. Below are some valuable resources which could help to add some advance elements..

* [***My Github profile README***](https://github.com/pratikjagrut/pratikjagrut)
    
* [***Create badges: shields.io***](https://shields.io/)
    
* [***Awesome github profile readme templates***](https://github.com/abhisheknaiidu/awesome-github-profile-readme)
    
* [***Next Level GitHub Profile README by codeSTACKr***](https://www.youtube.com/watch?v=ECuqb5Tv9qI&t=716s)
    
* [***UPDATE: Next Level GitHub Profile README (NEW) | GitHub Actions | Vercel | Spotify by codeSTACKr***](https://www.youtube.com/watch?v=n6d4KHSKqGk&t=181s)
    

***I hope you found this blog helpful and thank you for reading it please give your feedback in the comment section below.***