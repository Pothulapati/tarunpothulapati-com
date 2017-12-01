Title: Setting up a blog with Wyam and Github Pages
---
 I've always wanted a blog and everytime I chose the *hard-way* as I wanted to manage the whole site myself on a Windows server (*because .Net Framework*) and then with the advent of .Net Core I moved onto a linux server which decreased my costs slightly but I wasn't happy with either of these options as my Azure credit used to finish before the end of the month resulting in interruption of services(including my Blog).
  
  Now here I'am with a blog which costs only my time (which I have in plenty atleast for now).

### Wyam

>[Wyam](https://wyam.io/) is a simple to use, highly modular, and extremely configurable static content generator that can be used to generate web sites, produce documentation, create ebooks, and much more.

It let's you convert your dynamic sites into static sites which can then be **freely** hosted on services like [Github Pages](https://pages.github.com/),[Netlify](https://www.netlify.com/). You can also convert Asp.net MVC sites,Razor Views,etc to static sites using Wyam.

You can check the wyam documentation [here](www.wyam.io/docs)
### Github Pages
>[GitHub Pages](https://pages.github.com/) is a static site hosting service.GitHub Pages is designed to host your personal, organization, or project pages directly from a GitHub repository. 

It let's you host static content ranging anything from websites to documentation along with a custom domain.
## Wyam and Github Pages to rescue

Now let's start setting up a blog with Wyam using Blog recipe and CleanBlog theme.
Download the latest  version of Wyam by going [here](https://github.com/Wyamio/Wyam/releases). Once you complete installing Wyam, Make sure you add the environment variables for the commands to work.

Now Run the following command in the directory you want to scaffold the Blog template with basic files a blog usually contains.
```
wyam new --recipe Blog
```
Now you can see the following files in the folder.

![](../images/directory.png)


The input directory contains the content files.These files can be markdown files,razor files,etc.If you want add any links in the header, then you would add those files directly into the input folder and for the posts you need to add the markdown files into the posts folder.

These files act as input to wyam which converts these files into static files along with a theme you want. Now Let's convert our markdown,razor files into static files.

```
wyam --recipe Blog --theme CleanBlog
```

**BOOM!** Now, You can see a output folder with all the Static content files like HTML,CSS,etc.

You can know checkout the site you created using the command ```wyam preivew```.

Now, We have a site running locally. Push the site to a github repository(public only) and setup the github pages by going into the repo settings. There are also further instructions available to setup a custom domain in the settings page.

I know its hard to make changes  and then run the wyam command then push to the github repository everytime you make a change.

So,Let's Automate the process. We can use appveyor to read the input from a github branch, let it run wyam on it and then push the output to a github branch which has github pages setup on it.

Looks Cool, Dosen't it? Here is the Appveyor script to set it up.
<script src="https://gist.github.com/Pothulapati/2f4c6b0c8b7c0063df2586180ef2c362.js"></script>

Replace the 

The Above script does the following
* Gets the input files from the branch specified.
* Gets and Installs the latest version of wyam.
* Runs wyam on the specified branch.
* Saves the output of wyam to the specified branch ( which is published by github pages).


























