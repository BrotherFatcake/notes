//Clone repo and then redirect push location to own repo

-Go to dir that repo should be created.

-git clone <enter git SSH URL here>

-cd <dir name of cloned repo>

-create new repository on GitHub

-set new origin(push) location
    git remote set-url origin <gitHun SSH URL of new repo you created on GitHub>

-Verify fetch/push locations are correct
    git remote -v

-Push cloned repo to new repo location you created 
    git push








//Fork, then clone and update repo to provide PULL REQUEST from Command Line


create fork on github (I didn't find a command line way to fork)


git clone git@github.com:BrotherFatcake/seattle-metropolitans-interview.git

git checkout -b tbChange


echo "Hello World" > README.md

git add .

git commit -m 'add readme'

git push -u origin tbChange

git request-pull -p origin/master git@github.com:BrotherFatcake/seattle-metropolitans-interview.git tbChange

------------------

Based on what I've found the output below would be sent to the maintainer so they can fetch my changes.  

I also entered a pull request on the web - https://github.com/Thinkful-Ed/seattle-metropolitans-interview/pull/180    

---OUTPUT of command line request-pull --- 

The following changes since commit 3e5017dc20187bee6635c6ac420b6c47a47620eb:

  initial commit (2017-11-08 17:33:09 -0500)

are available in the git repository at:

  git@github.com:BrotherFatcake/seattle-metropolitans-interview.git tbChange

for you to fetch changes up to d5cdb8e8e9beddd927adfaf9c92bbdcfe8101f34:

  update README (2018-06-09 11:22:54 -0500)

----------------------------------------------------------------
BrotherFatcake (2):
      add README
      update README

 README.md | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 README.md

diff --git a/README.md b/README.md
new file mode 100644
index 0000000..980a0d5
--- /dev/null
+++ b/README.md
@@ -0,0 +1 @@
+Hello World!




API hack

could use to call API, then promise to come back to it once API responds the app can move on to other things.  Watch for the 200 status 


linter - pre-defined style guide - self defined > kind of like code equiv of spellcheck

