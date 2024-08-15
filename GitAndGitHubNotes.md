# Learn Git and GitHub

## Install Git
On Debian based distros use:
```bash
sudo apt install git
```

## Linking local git to Github:
We start by linking the local user (us) to GitHub.
```bash 
git config --global user.name "My Username"
git config --global user.email "MyEmail@example.com"
```
- If we opted to use the Private Github email Address the second command would look like:
```bash 
git config --global user.email "123456789+odin@users.noreply.github.com"
```
- Github recently changed the name of the **master** branch to **main** so we need to make the changer on our system.
```bash 
git config --global init.defaultBranch main 
```
- We can enable colorful output with 
```bash
git config --global color.ui auto.
```
- We can also set our default **Branch Reconciliation Behavior** to **merging** 

```bash
git config --global pull.rebase false
```
- We can verify user info with 
```bash
git config --get user.name 
git config --get user.email
```
### Changing the default text editor
```bash
git config --global core.editor "vi"
```
 
## Creating an ssh key
An SSH key with an Ed25519 algorithm can be used to uniquely identify our computer on github when we upload our code instead of typing our password every time. 
- We first need to verify if we already have an ed25519.pub public key in our computer

```bash
ls ~/.ssh/id_ed25519.pub
```
- If we don't, we create one with the following command
```bash
shh-keygen -t ed25519
```

### Linking the SSH key to our GitHub
with the cat command we can highlight our key and copy it to our github account.
``` bash
cat ~/.ssh/id_ed25519.pub
```
Then go to ADD SSH Key on our GitHub account and paste the key.
### Testing the key
we can test our key with the following command
```bash
shh -T git@github.com
```
If everything goes well we should receive a message greeting us.

## Using git
Create a new git repository _git-test_ with a readme file.
- Clone the git repo by selecting the ssh option.

```bash
git clone git@github.com:ichahiin/git-test.git
```
- To test that our command succeeded, _cd_ into the directory and type:

```bash
git remote -v
```
it will return the provenance of the cloned file.

### Getting familiar with the git Workflow
- Create a file name _hello.txt_ and type: 
```bash 
git status
```
- The _hello.txt_ file appears in red as it is an **Untracked file** and type:
```bash
git add hello.txt
```

This will add our file to the _staging_ area in our file, think of the _staging_ area as a waiting room for our changes until we _commit_ them.
we can verify that by typing _git status_ again.
- now we can _commit_ our file with:
```bash
git commit -m "We've added hello.txt"
```
- We can see a list of our commits with:
``` bash
git log
```

### Modifying files
after modifying a file we use the git add command again.

## Pushing changes to GitHub
We can publish or "push" our _main_ branch to github with the following command:

```bash
git push origin main
```

