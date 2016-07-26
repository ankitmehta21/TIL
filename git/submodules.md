# Working with git submodules

I've worked with git enough to understand what a submodule is and why I would want to use one, but I ran into trouble every time I tried adding a submodule to a project.
This last weekend, however, I finally had some success with submodules while configuring my [pelican blog](blog.harterrt.com].

Proper credit where due, [this article by Christian Long](http://www.christianlong.com/blog/more-on-pelican-themes.html) was my primary source.

```bash
# Add a submodule
git submodule add {{repo}} {{local_directory}}
git commit -am "Adding {{repo}} submodule"

# View submodules
git submodule status
git submodule status --recursive  # If the submodule has submodules, like pelican-themes

# Initialize recursive submodules as desired
git submodule init {{sub-submodule}}
git submodule update

```

Voila. Now I dont't even know why I thought that was difficult.
