Setting up a Development Environment
Basics of Git

## Git

It is one of the most widely used version control system in the world, it is developed by the same guy who built the Linux operating system kernel ( Linus Torwarlds ) and it is an essential prerequisite for almost all software development related jobs out there.

### Installing git

This is already covered in the previous sections, if you still haven't configured it view [this article](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) to get you up to speed

### Learning Git

Assuming the same example as the section above, we'll see how git would have helped in your group project.

Let's assume for now that just alone is working on this project, and you wanted to track the changes you made to your report as you complete it.

To start with let's create a new folder called `learning-git`, you can create this folder wherever you want.

Now that you have your folder created, open up a terminal/command prompt and navigate to your current directory, once you are inside your newly created folder run the following command.

```git
git init
```

\*_if this command returns an error then git has not been configured in your system, look back at earlier sections and make sure that git is installed correctly._

This command creates a new git _Repository_, a Repository is similar to a project( with version control ), now your unversioned project is a git repository.  
if you look closely you will see a new hidden folder `.git` created inside your folder, git uses this folder for all its bookkeeping, unless you are a git wizard don't try to edit anything in there.

Now we have a really cool version-controlled project but its kinda empty, so let's create a new file in it, you can use your favourite code editor or Linux commands to create a new file in your folder. for the sake of consistency let's create a file called `cats.txt` containing the following text

`Cats: A Brief Summary`

Now that you have created a new file, we can take a look at how we can track its changes with git.

### Working Directory, Staging Area and Repository

A git development environment consists of three sections

1. **Working Directory**  
   This is simply the current state of files and folders inside your current folder, these changes are **_not_** yet recognized by git and are **_not_** tracked yet.
2. **Staging Area**  
   This is a temporary location for your files before they are saved to a repository. You can add more than one file to the staging area.
3. **Repository**  
   This is where your actual work and its history lies. You can sync repositories to other computers and expect the same copies to be present.

To view the current status of our git repository run the following command

```git
git status
```

Once you run the command, git will inform you that you have created a new file. ( the file is still in the working directory )

To stage the file with git we can run the following command,

```git
git add cat.txt
```

This command will add the file `cat.txt` into the git staging area, alternatively, you can do `git add .` which will add all files in your folder into the git staging area (when you don't want to manually specify each file)

Once the files have been staged, they have to be moved to the repository, this is called a commit. Commits are usually a logical chunk of change, you don't have to commit for every change you make to the file. In our example, you can commit the file once you have completed a section of the report or such. Commits create snapshots of the repository, you can restore your repository to any commit at any time.

Commits are ideally associated with a commit message, the commit message makes it easier to understand the intent of the change, this is incredibly useful if you wanted to restore the repository into an earlier commit you made.

The following command is used to create a commit in git

```git
git commit -m "Started to create a report on cats"
```

This command will take all the files in staging and commit them to the git repository. Once you have committed, running `git status` again will let you know that you no longer have any changes (Your working directory and the git repository are in sync now). Committing clears the staging area so that you can start working on your new changes.

To view all your past commits you can use the log command

```git
git log
```

( use q to exit the command )

This command will show all the commits made in your repository along with who made the change.

Now, make some changes to your report. You can add new content, delete existing content or even create new files. Once you are done, run `git diff` this will show all changes you have made in your working directory.

Now you have a basic idea of what git is and how to perform basic actions in git. Try making more changes in your folder and try to record those changes with commits to get familiar with git.

# Introduction to GitHub

## Collaboration with Git

Git enables multiple developers to work on the same project simultaneously. GitHub (different from Git) provides online repository hosting for team collaboration.

### Key Benefits
- **Remote Access**: Team members can access repositories online
- **Offline Work**: Continue development without internet (sync when needed)
- **Version Control**: Track changes across the entire team

## Creating a GitHub Repository

1. **Sign up/Login** to GitHub
2. **Click "New repository"** (under the + icon)
3. **Name your repository** (use hyphens instead of spaces)
4. **Choose visibility**: Public (anyone can see) or Private (invite-only)
5. **Uncheck** "Add a README file" and other options
6. **Create repository**

## Connecting Local Repository to GitHub

### Get Repository URL
Your GitHub repo URL follows this format:
```
https://github.com/your_username/repository_name
```

### Link Repositories
```bash
git remote add origin https://github.com/your_username/repository_name
```
- `origin` is the standard name for your remote repository
- You can use any name, but `origin` is conventional

## Git Branches Basics

Branches allow parallel development without affecting main code:

- **Default branch**: Usually `main` (formerly `master`)
- **Check current branch**: `git status` (shows current branch in first line)
- **Create branches** for features, bug fixes, or experiments

## Syncing with Remote Repository

### Push Local Changes to GitHub
```bash
git push origin main
```
- `origin`: Remote repository reference
- `main`: Branch name

### Pull Changes from GitHub
```bash
git pull origin main
```
- Downloads and merges remote changes

## Merges and Conflicts

### Automatic Merges
Git automatically merges non-conflicting changes from different branches.

### Conflict Resolution
When multiple developers edit the same file:
1. Git marks conflict areas in the file
2. Manually edit file to choose desired changes
3. Commit the resolved version

## Forking

**Forking** creates a personal copy of someone else's repository:

- **Purpose**: Experiment without affecting original
- **Use case**: Propose changes to open-source projects
- **GitHub UI**: Click "Fork" button on any repository

## Pull Requests

**Pull Requests (PRs)** propose merging your changes:

- **Process**: Request to merge your branch/fork into main repository
- **Benefits**: Code review, discussion, and quality control
- **Workflow**:
    1. Create PR from your branch/fork
    2. Team reviews and discusses changes
    3. Make additional commits if needed
    4. Merge when approved

## Ignoring Files

### .gitignore File
Prevent tracking unwanted files (configs, secrets, build files):

1. **Create** `.gitignore` in repository root
2. **Add patterns** for files to ignore:
   ```
   *.log
   .env
   __pycache__/
   ```
3. **Template**: Use [Python .gitignore template](https://github.com/github/gitignore/blob/main/Python.gitignore)

## Git Best Practices

### Essential Tips
- **Experiment safely**: Git is forgiving, but backup important work
- **Never copy-paste** commands without understanding them
- **Meaningful commits**: Write clear commit messages
- **Descriptive branches**: Use names like `feature/user-auth` or `bugfix/login-error`

### Learning Path
- **Practice**: Complete the [Git-it Guide](http://jlord.us/git-it/)
- **Contribute**: Find open-source projects and submit pull requests
- **Keep learning**: Git has endless depth - explore as you go

---

**Remember**: Git mastery comes from practice. Start with simple repositories and gradually tackle complex collaboration scenarios.


Command lines


--
