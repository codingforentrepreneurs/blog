---
title: Terminal Prompt Formatting
slug: mac-linux-terminal-prompt-formatting

publish_timestamp: Nov. 17, 2018
url: https://www.codingforentrepreneurs.com/blog/mac-linux-terminal-prompt-formatting/

---


When you open up `Terminal` for the first time, you'll probably want to format it to suit your needs. This post shows you some formatting tips.

> Since `Terminal` is the command line utility for **Mac** and **Linux**, this guide is just for **Mac** and **Linux** users. 

Adjusting your terminal prompt's default message is simple. The default I use is:

```
# ~/.bash_profile
export PS1="$ "
```
To me, it's clean and removes uncessary information just like most guides you'll see online. 

```console
$ 
```

##### Example with a virtual environment activated
> If you're new to terminal, you can ignore this for now.
```console
(myvenv) $ 
```


<br/>
<hr/>

### Editing
To change your default terminal prompt from

``` console
justins-mbp:~ jmitch$ 
```

The only command you need is `export PS1=` and then set the string with the commands below. 

If you want to keep a new format, just edit your `.bash_profile` with:

```console
justins-mbp:~ jmitch$ nano ~/.bash_profile
```
> _Permission error_? You might need to use `sudo` prior. So the comamnd is `sudo nano ~/.bash_profile`

Once you open up `.bash_profile` just add the line

```
export PS1="$ "
```
Replace the string `"$ "` with any command you see below.
<br/>
<hr/>

### Commands

`\u` – Current user

`\W` – Current working directory 

`\w` – Current working directory including path

`\d` – Date

`\t` – Time

`\h` – Hostname

`\#` – Command number

<br/>
<hr/>


### Examples

##### Current User `\u`
```
# ~/.bash_profile
export PS1="\u $ "
```

Result
```console
jmitch $ 
```

<hr/>
<br/>


##### Command Number `\#`
```
# ~/.bash_profile
export PS1="(\#) $"
```

Result
```console
(1) $ 1 $ cd desktop
(2) $ ls
Screen Shot 2018-11-17 at 9.35.09 AM.png
todo.md
(3) 
```
<hr/>
<br/>

##### Date + Time and User `\d:\t`
```
# ~/.bash_profile
export PS1="\d:\t - \u $ "
```

Result

```console
Sat Nov 17:09:16:28 - jmitch $ 
```

<hr/>
<br/> 

##### Current directory
```
# ~/.bash_profile
export PS1="\W $ "
```

Result
```console
desktop $ 
```

<hr/>
<br/>

##### Current directory w/ path
```
# ~/.bash_profile
export PS1="\w $ "
```

Result
```console
~/desktop $ 
```
