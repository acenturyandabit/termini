# Stack

As a C++ software developer / databricks backtest runner, I occasionally have long-running tasks that take unpredictable amounts of time. I want to have an automated way of alerting me when those tasks are complete.

![Screenshot of the app showing long running tasks](screenshot.png)

## Installation

Requirements: node/npm on your system

```bash
# Clone this folder to somewhere out of the way.
git clone $THIS_REPO
cd stack

# Install dependencies
npm install .

# Add the `bashrc_supplement` to your .bashrc; restart shells or source .bashrc
cat bashrc_supplement >> ~/.bashrc; source ~/.bashrc

# Run the server
tsx ./server.ts

# Optional: You may want to use a https server with a self-signed certificate as this will allow userscripts from https sites to send requests to your server. To do this:
openssl req -x509 -newkey rsa:4096 -keyout server.key -out server.crt -nodes -subj "/C=XX/ST=StateName/L=CityName/O=CompanyName/OU=CompanySectionName/CN=CommonNameOrHostname"

# And so you would instead run:
HTTPS=true ./server.js
# Also don't forget to edit the bashrc_supplement to use https instead of http.

```

## Usage

1. Open the URL according to the console output.
2. For new shells, run `stack-register-thread TASKNAME` to register a shell to a task. (You might want to alias this, its pretty long)
3. Start a long running task on your shell that you've registered. Stack will be alerted to its presence.
4. When the task is done, Stack will let you know.
5. Ack that the task is done using `stack-ack` in the same shell as the process, then keep doing your work.

## Advanced usage

From the web UI, you can click the '^' button on each task to rearrange its priority. A higher priority task that is marked 'YOU_COULD_BE_DOING' takes precedence over a lower task that is marked 'COMPUTER_DOING', so you will always be notified when your highest priority tasks are available to do.

## How it works

There are three actors in our model: You, the Computer, and Stack. You have a number of threads of work. Your job is to do your most urgent thread; but sometimes you ask the Computer to do your thread, e.g. compile a piece of code. When this happens, you will go work on another, less urgent thread. When the Computer has finished on your most urgent thread, the Stack will notify you that you are able to resume your most urgent thread.

(Note that the Computer can represent a build job, a cloud runner, or even another person - anything that is blocking your highest priority thread.)

A thread can be in one of four states: You're working on it, You could be working on it, the Computer is working on it, or Dead (done or cancelled).

## What Stack actually keeps track of

Stack can't actually keep track of threads-of-work; instead it can only keep track of PIDs on the system (of bash shells), or browser windows.

### Bash shells / PIDs

You should register bash shells by typing `stack-register-thread TASKNAME` to associate a bash shell to a thread. Subsequently, Stack, via `server.js`, will check whether or not the shell has any child processes. If any shell associated with the thread has a child process, Stack will assume that the Computer is working on the thread. If all shells associated with the the thread dies, Stack will assume the thread is dead.

The `server.js` will give the frontend a list of tasks, their state, and any metadata for that state (e.g. process names).

The frontend keeps a record of thread priority, and will display in large font the highest priority thread that is you're-working-on-it. This is stored in the frontend localstorage, because I can't be bothered to make a way for the server to store it right now.

### Browser / external tasks

Use the `userscript.ts` and Tampermonkey or your favourite userscript runner to setup browser-forwarded task states. This works great with long running searches from your company's data insights tooling.
