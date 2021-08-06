---
layout: post
title: Rust SSH
subtitle: Learning Rust by building a CLI tool
image: /assets/img/rusty_logo_small.jpeg
cover-img: /assets/img/rusty_logo.jpeg
thumbnail-img: /assets/img/rusty_logo_small.jpeg
share-img: /assets/img/rusty_logo.jpeg
tags: [rust, rust basics]
---

Creating small CLI tools is a fun way to get more familiar with a programming language. If you are coming from an infrastructure background, a CLI tool that you can use to send commands to devices/servers might be considered a neat starting point getting into Rust. This is how I started off learning Python, by writing small things that were usefull in a context that I was familiar with. Back then, I used `argparse`, `getpass` and `netmiko`. Starting with Rust is pretty daunting, and I have found that using similar tactic used to learn Python can also be applied when learning Rust.

In this article, I explore a tiny bit of Rust by building a CLI tool that will let you log into a router or a server and issue a command over SSH. I hope the examples here can offer you a nice running start exploring Rust.


### Running a CLI command over SSH

In case you do not have Rust installed, one easy way of going about things is to grab the rust Docker container:

<pre>
docker pull rust
docker run --name='ru' --hostname='ru' -di rust /bin/sh
docker exec -it ru bash
apt-get update
apt-get install vi
</pre>

<p style="font-size:11px;">The Rust image is 1.22GB 😳</p>

After this, we use `cargo new` to start a new project:

<pre>
cargo new ssh
cd ssh/
vi Cargo.toml
</pre>

Now we put in place the following file:

```toml
[package]
name = "ssh_example"
version = "0.1.0"
edition = "2018"

[dependencies]
ssh2 = "0.9"
```

This let's us pull in the `ssh2-rs`, which offers Rust bindings to libssh2, the ssh client library written in C.

Now we put in our first example script. We edit the `main.rs` file in the `src` directory:

<pre>
vi src/main.rs
</pre>

Then we put in the following:

```rust
use ssh2::Session;
use std::io::prelude::*;
use std::net::TcpStream;

fn main() {
    let tcp = TcpStream::connect("192.168.1.1:22").unwrap();
    let mut sess = Session::new().unwrap();
    sess.set_tcp_stream(tcp);
    sess.handshake().unwrap();
    sess.userauth_password("username", "password")
        .unwrap();
    let mut channel = sess.channel_session().unwrap();
    channel.exec("show version").unwrap();
    let mut s = String::new();
    channel.read_to_string(&mut s).unwrap();
    println!("{}", s);
    channel.wait_close().ok();
    println!("{}", channel.exit_status().unwrap());
}
```

This will use SSH to connect to a system configured with IP address `192.168.1.1` and issue the `show version` command.

After putting in the file, we can use `cargo run` to run the code:

<pre>
root@ru:/ssh# cargo run 

    Finished dev [unoptimized + debuginfo] target(s) in 0.83s
     Running `target/debug/ssh`
Arista DCS-7050TX-64-R
Hardware version:    01.01
Serial number:       KLF80800800
System MAC address:  001c.84ec.cb72

Software image version: 4.20.12M
Architecture:           i386
Internal build version: 4.20.12M-11527863.42012M
Internal build ID:      3a8329a8-af7b-4a7e-ae1c-cad006c5540d

Uptime:                 114 weeks, 1 days, 7 hours and 2 minutes
Total memory:           3818208 kB
Free memory:            2258276 kB


0
</pre>


### Collecting user input

We can add some CLI options to running this script by using `structopt`. First, change our `Cargo.toml` to the following:

<pre>
[package]
name = "ssh_example"
version = "0.1.0"
edition = "2018"

[dependencies]
ssh2 = "0.9"
structopt = "0.3.21"
</pre>

This brings in the [structopt](https://docs.rs/structopt/0.3.22/structopt/) crate and let's us define some arguments for our CLI script. In case you are familiar with Python, think `argparse`.

Let's first create a script that only collects the arguments:

```rust
use structopt::StructOpt;

#[derive(Debug, StructOpt)]
#[structopt(name = "structopt example", about = "using structop")]
struct Args {
    #[structopt(short = "h", long)]
    host: String,
    #[structopt(short = "c", long)]
    command: String,
    #[structopt(short = "u", long)]
    username: String,
}

fn main() {
    println!("Structopt example");
    let args = Args::from_args();
    println!(
        "{:?}\nHost: {}\nCommand: {}\nUsername: {}",
        args, args.host, args.command, args.username
    );
}
```

We can run the above like so:

<pre>
root@ru:/ssh# cargo run -- -h 192.168.1.50  -c 'show version' -u said
   Compiling ssh_example v0.1.0 (/ssh)
    Finished dev [unoptimized + debuginfo] target(s) in 1.21s
     Running `target/debug/ssh_example -h 192.168.1.50 -c 'show version' -u said`
Structopt example
Args { host: "192.168.1.50", command: "show version", username: "said" }
Host: 192.168.1.50
Command: show version
Username: said
</pre>

Looks good. 

Though, we would also need to collect the password. For that, we include the `rpassword` crate by adding the following to our `Cargo.toml`:

<pre>
rpassword = "5.0"
</pre>

The `rpassword` crate allows us to ask users for a password without echoing it to screen and use that in our script. 

To collect the arguments as well as the password, we now have the following:

```rust
extern crate rpassword;
use rpassword::read_password;
use structopt::StructOpt;

#[derive(Debug, StructOpt)]
#[structopt(name = "structopt example", about = "using structop")]
struct Args {
    #[structopt(short = "h", long)]
    host: String,
    #[structopt(short = "c", long)]
    command: String,
    #[structopt(short = "u", long)]
    username: String,
}

fn main() {
    println!("Enter your password: ");
    let password = read_password().unwrap();

    println!("The password is: '{}'", password);
    println!("Structopt example");
    let args = Args::from_args();
    println!(
        "{:?}\nHost: {}\nCommand: {}\nUsername: {}",
        args, args.host, args.command, args.username
    );
}

```

When we run this, we see the following:

<pre>
root@ru:/ssh# cargo run -- -h 192.168.1.50  -c 'show version' -u said
    Finished dev [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/ssh_example -h 192.168.1.50 -c 'show version' -u said`
Enter your password: 

The password is: 'mypassw0rd'
Structopt example
Args { host: "192.168.1.50", command: "show version", username: "said" }
Host: 192.168.1.50
Command: show version
Username: said
</pre>

Great! Now we can move on and combine the two.

### Tying things together

We create the following `Cargo.toml`:

<pre>
[package]
name = "ssh_example"
version = "0.1.0"
edition = "2018"

[dependencies]
ssh2 = "0.9"
structopt = "0.3.21"
rpassword = "5.0"
</pre>

Then we update the SSH script:

```rust
use ssh2::Session;
use std::io::prelude::*;
use std::net::TcpStream;
extern crate rpassword;
use rpassword::read_password;
use structopt::StructOpt;

#[derive(Debug, StructOpt)]
#[structopt(name = "structopt example", about = "using structop")]
struct Args {
    #[structopt(short = "h", long)]
    host: String,
    #[structopt(short = "c", long)]
    command: String,
    #[structopt(short = "u", long)]
    username: String,
}

fn main() {
    println!("Enter your password: ");
    let password = read_password().unwrap();
    let args = Args::from_args();
    println!(
        "Running command {} against host {}:\n",
        args.command, args.host
    );
    let tcp = TcpStream::connect(args.host + ":22").unwrap();
    let mut sess = Session::new().unwrap();
    sess.set_tcp_stream(tcp);
    sess.handshake().unwrap();
    sess.userauth_password(&args.username, &password).unwrap();
    let mut channel = sess.channel_session().unwrap();
    channel.exec(&args.command).unwrap();
    let mut command_output = String::new();
    channel.read_to_string(&mut command_output).unwrap();
    println!("{}", command_output);
    channel.wait_close().ok();
    println!("{}", channel.exit_status().unwrap());
}
```


When we run the above code, we get the following:

<pre>
root@ru:/ssh# cargo run -- -h 192.168.1.50  -c 'show version' -u said

    Finished dev [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/ssh_example -h 192.168.1.50 -c 'show version' -u said`
Enter your password: 

Running command show version against host 192.168.1.50:

Arista DCS-7050TX-64-R
Hardware version:    01.01
Serial number:       KLF80800800
System MAC address:  001c.84ec.cb72

Software image version: 4.20.12M
Architecture:           i386
Internal build version: 4.20.12M-11527863.42012M
Internal build ID:      3a8329a8-af7b-4a7e-ae1c-cad006c5540d

Uptime:                 114 weeks, 1 days, 7 hours and 2 minutes
Total memory:           3818208 kB
Free memory:            2258276 kB


0
</pre>

Instead of targeting a router, we can also play around targeting Linux servers:

<pre>
root@ru:/ssh# cargo run -- -h 10.0.0.1  -c 'ls -ltr' -u said           

    Finished dev [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/ssh_example -h 10.0.0.1 -c 'ls -ltr' -u said`
Enter your password: 

Running command ls -ltr against host 10.0.0.1:

total 25032
-rw-r--r--.  1 said UnixUsers 25627989 May  3 11:11 Python-3.9.5.tgz
drwxr-xr-x. 16 said UnixUsers     4096 Jun  8 06:38 Python-3.9.5

0
</pre>


### Wrapping up

We created a small CLI tool that allows us to send a command over SSH to a server or a router. In the examples, I send a command to Linux server and an Arista switch. I did not get into all the specifics and this is not something that is ready for production. For one, I do not properly handle the `Result`, instead I used `unwrap()` everywhere. The main point in this article was to give you a running start sending CLI commands to systems over SSH. I hope this gives you a nice starting point.

