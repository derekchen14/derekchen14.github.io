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
This is the equivalent statement that gets run in the background:
wget -r -nH -P docs -E -T 2 -np -k http://localhost:2368/

Then, you can visit the root path to view the blog.
Most likely, you will want to edit the blog which is located at:

  http://localhost:2368/ghost/

## Remote Deployment

To deploy your updates to the web you need to add changes to `git`, commit, and then push.

# Deprecated
The original files are located at:
 feature-engineering.ghost.2020-08-12-01-10-45.json
but no longer apply.  This is just for record keeping.



