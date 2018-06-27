# local-jekyll-docker

An example repository of running a Jekyll based blog locally with Docker.
A related blog post can be found [here][blog]. Note that this setup is not intended for production
use but helps to play around with Jekyll in your local environment.

## Running the example blog locally

The blog source files are located in the `blog` folder. You can serve them using the official 
[Jekyll Docker image][jekyll-docker].

First cd into the `blog` folder. Then type:
```
docker run --rm --volume="$PWD:/srv/jekyll" -p 4000:4000 -it jekyll/jekyll:3.8 \
  bash -c "bundle install && bundle exec jekyll serve -H 0.0.0.0"
```

This uses the `jekyll/jekyll:3:8` Docker image. The command will download and install the needed 
Gems (Ruby libraries) to the container and finally start the server.

Now type `localhost:4000` to your browser's address bar and you should see the blog. Any changes 
that you make to the files in the `blog` directory are automatically updated to the blog (except
for the `_config.yml` which requires a restart of the server).

### Docker run argument explanations

Some explanations for the `docker run` command above:
- **--rm**: this removes the container after the server is stopped
- **--volume**: this tells which local folder to bind mount to the container. Here the current working 
  directory is mounted to `/srv/jekyll` in the container.
- **--p**: this forwards the host machines port to the container port, here 4000 to 4000 because
  Jekyll by default starts the server to port 4000.
- **--it**: run in interactive mode so that you can pass on commands to the container 
  (mainly stop the served with CTRL-C)
- **jekyll/jekyll:3.8** is the Docker image name (here `jekyll/jekyll`) and version (here 3.8)
- **bash -c**: run a command using the bash shell binary in the container. I use this to enable
  chaining of commands with &&.
- **bundle install**: install all the gems and their dependencies in the Gemfile
- **bundle exec**: run executable in the context of the current Gemfile.
- **jekyll serve -H 0.0.0.0** run the server on 0.0.0.0. The default host would be localhost but
  it would not be reachable outside the container (in this case from our machine).

### Speeding up the starting of the server

Most of the time in starting the server with the command above is spent in loading and 
installing the gems. The repeated installation can be avoided by making a new image that already 
contains the needed gems. We could use a `Dockerfile` for it but in this case it's straightforward 
to create it from the command line. Assuming we are in the `blog` folder, type:

```
docker run --name blog_server --volume="$PWD:/srv/jekyll" -it jekyll/jekyll:3.8 \
  bundle install
```

Notice that we removed the `--rm` argument which would remove the container after execution. Instead 
we added `--name` to give the container a name `blog_server`. Now typing `docker ps -a` 
should show you a container with that name. It has all the gems installed
and the final step is to make an image of it with `docker commit`:

```  
docker commit blog_server blog_server
```

Now we have an image called `blog_server` that has the needed gems installed. 
You can see this image listed by typing `docker images`. 
The server can now be ran using our new image with

```
docker run --rm --volume="$PWD:/srv/jekyll" -p 4000:4000 -it blog_server \
  bundle exec jekyll serve -H 0.0.0.0
```

You will notice that it starts in an instant. As long as you keep the image in your computer you can 
use it to serve the blog. If you at some point update your bundle (dependencies) you can recreate
the image with the steps above.

## Creation of the example blog files

The Jekyll blog files in this repository were created using the official 
[Jekyll Docker image][jekyll-docker].

First create a blog folder to your project directory for storing the blog files:

`mkdir blog`

Now create the Jekyll blog files with the Jekyll Docker image (I used `jekyll/jekyll:3.8`) by 
running: 

```
docker run --rm --volume="$PWD/blog:/srv/jekyll" -it jekyll/jekyll:3.8 \
  jekyll new . --skip-bundle
```

The command mounted your `blog` directory to `/srv/jekyll` in the container and then created the 
blog files with `jekyll new`. They should now appear in your `blog` folder.


[blog]: https://jlintusaari.github.io/2018/03/github-hosted-jekyll-blog-custom-theme/
[knitr]: https://yihui.name/knitr/
[GitHub]: https://github.com
[Jekyll]: https://jekyllrb.com/
[jekyll-in-github]: https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/
[github-pages]: https://pages.github.com/
[gh-themes]: https://pages.github.com/themes/
[whiteglass]: https://github.com/yous/whiteglass
[remote-theme]: https://github.com/benbalter/jekyll-remote-theme
[issue]: https://github.com/yous/whiteglass/issues/13
[bundler]: https://bundler.io/
[jekyll-docker]: https://hub.docker.com/r/jekyll/jekyll/

