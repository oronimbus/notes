# How to create a new repo in Github

1) For Windows, right-click on folder and Git Bash. For Mac just use terminal and cd into folder and then type:
```
git init
```
2) Add origin (ignore if already available) via copy-pasting .git project URL
```
git remote add origin https://...../project.git
```
3) Check status for files to commit:
```
git status
```
4) Add files to be submitted:
```
git add -A
```
5) Commit files with a comment: 
```
git commit -m "initial commit"
```
6) Push changes/files to git, -u origin can be omitted if it's not the first commit.
```
git push -u origin 
```