# [Room 404](https://tryhackme.com/room/hh-room404-804573bf)

- Hackers holidays 2026 day 2 challenge.  it is a web challenge lets dive into it.

![screenshot](../data/R4041.png)

- Start the machine and visit the site in a browser.
![screenshot](../data/R4042.png)
- As mentioned in the lab decsrition lets perform directory enumeration . 
![screenshot](../data/R4043.png)
- We found a intresting directory .git . 
![screenshot](../data/R4044.png)
- we foundan entire git repository. Lets re construct the entire repository in our system so we can access it via terminal.
- i am going to use a tool called git-dumber. you can install it using pip.
```
pipx install git-dumper
```

![screenshot](../data/R4045.png)

- now get into the git_dir directory and start inspecting the git repository with command line .

```
git log --stat
```
![screenshot](../data/R4046.png)

- Copy the hash and use the below command 
```
git show <hash>
```
![screenshot](../data/R4047.png)

**Flag: THM{byt3_l0tus_n3v3r_f0rg3ts}**
