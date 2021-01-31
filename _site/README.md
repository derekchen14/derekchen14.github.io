# Starting Up
To get started, you should be in the ghost folder which is at:

  /Documents/active/morethanoneturn/ghost/

## Local Development
Then, you can use these commands operate the blog
  - `ghost stop` to stop Ghost
  - `ghost start` to start Ghost
  - `ghost log` views logs
  - `ghost ls` to list all running Ghost blogs
In particular `ghost start` will start a local instance of the blog.
Then, you can visit the root path to view the blog. Most likely, you will want to edit the blog which is located at:

  http://localhost:2368/ghost/

Once you are done editing, you can publish your changes to the local version.

## Remote Deployment
To deploy your updates to the web, you start by generating the web files.
With the local server still on, you can run:

   wget -r -nH -P docs -E -T 2 -np -k http://localhost:2368/

This will re-generate all the files in ghost directory, including a new 'docs' folder.
Look in this folder and copy over any new blog posts into the parent folder.
Then, as usual you will need to add changes to `git`, commit, and then push.
This will update derekchen14.github.io, which is connected to the custom URL through the CNAME file.
If all goes well, then pushing the changes will kick off the production script (on Github.io),
so you are all done!

# Deprecated
The original files are located at:
 feature-engineering.ghost.2020-08-12-01-10-45.json
but no longer apply.  This is just for record keeping.



