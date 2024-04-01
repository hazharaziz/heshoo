# Zangata #1 - Configure Docker Credential Helpers for Linux

The other day I was trying to pull an image from docker registry for the first time in my ``Linux Mint 21.3(Virginia)``. It didn't allow me to pull the image without being logged in to docker.

So, I tried to login with ``docker login`` command and looked for its [documentation](https://docs.docker.com/reference/cli/docker/login/) in the internet. It was totally straight-forward until I saw the [Credential stores](https://docs.docker.com/reference/cli/docker/login/#credential-stores) section in the mentioned documentation.

I read the section and decided to test it and apply it to my ``docker login`` process. Although it took half a day of my time, but it was so useful for me that I became familiar and also made a review of a couple of tools and concepts.

I don't want to repeat the documentation here again, but it will be better if you take a look at these links and read about them to get better understanding of what is going to happen:
- What is [GPG?](https://www.gnupg.org/gph/en/manual/c14.html)
- [pass](https://www.passwordstore.org/) - the standard unix password manager

## Why Credential helpers? 
When you ``docker login`` without using any credential helper tool, your docker authentication info gets stored in ``~/.docker/config.json`` like the following format:

```json
{
  "auths": {
    "https://index.docker.io/v1/": {
      "auth": "bXlfdXNlcm5hbWU6bXlfcGFzc3dvcmQ="
    }
  }
}
```

When you enter your username and password using in ``docker login``, docker encodes them and stores them in the above file. Now, take the value of ``auth`` field and decode it:

```bash
echo -n 'bXlfdXNlcm5hbWU6bXlfcGFzc3dvcmQ=' | base64 --decode
```

The result will include something like this:

```
my_username:my_password
```

The username and password that you entered for the login can be easily decoded from this file. So, anyone that has access to your system, legally or illegally, can find your docker credentials by a simple command.

That's where the ``Credential helpers`` come in and prevent your credentials to be exposed easily. 

## Generate a ``gpg`` key

- **Note**: Run ``gpg --list-keys`` in your terminal. If you have at least one ``gpg`` key, you can skip this step.

We should firstly generate a ``gpg`` key for encrypting our passwords. It is better to use full-generation and to be able to add a comment to our key too:

```
gpg --full-generate-key
```
You can use default options and generate the with your favourite name, comment and email.

Now, check out the list of your keys by running ``gpg --list-keys`` command and you should see an output like this:

```
/home/foo/.gnupg/pubring.kbx
-------------------------------
pub   rsa3072 2024-04-01 [SC]
      DA4098D273FDBC57914133992A5F6D20E8372B3B
uid           [ultimate] dockerfoo (some foo bar comment) <foo@bar.com>
sub   rsa3072 2024-04-01 [E]
```

In the above output, ``DA4098D273FDBC57914133992A5F6D20E8372B3B`` is the generated ``gpg-id``. Real name is ``dockerfoo``, comment is ``some foo bar comment`` and ``foo@bar.com`` is the email that we have entered during the process of full-generating our ``gpg`` key. 

That's it, our ``gpg-id`` is ready, let's move on!

## Initialize ``pass``

- **Note**: We assume that you have installed ``pass`` according to its documentation mentioned earlier.

Copy the generated ``gpg-id`` in the previous section and use it to initialize ``pass``:

```bash
pass init DA4098D273FDBC57914133992A5F6D20E8372B3B
```

Now, your ``pass`` is ready to work with its corresponding docker helper that we are going to set up in the next section. This tool has other capabilities that you can setup yourself according to its docs :)


## Setup Docker helper

According to Docker documentation for Credential stores, there are a couple of different [helpers](https://github.com/docker/docker-credential-helpers/releases) that we can use. This time we have chosen ``pass`` as the helper. So we should download it, make it executable and put it in our local binaries path for docker to find it. As a convention set by docker itself, we should add ``docker-credential-`` prefix to our helper binary.

### Download ``pass`` docker credential helper binary
From your ``$HOME`` directory, download pass credential helper using the following command:

```bash
wget -O docker-credential-pass https://github.com/docker/docker-credential-helpers/releases/download/v0.8.1/docker-credential-pass-v0.8.1.linux-amd64
```

- NOTE: The url for the helper may change based on the version or another helper. So be sure to check this [link](https://github.com/docker/docker-credential-helpers/releases) to get the right version of your helper. 


Make the downloaded ``docker-credential-pass`` executable:

```bash
chmod +x docker-credential-pass
```

Move ``docker-credential-pass`` binary to ``usr/local/bin``:

```bash
sudo mv docker-credential-pass /usr/local/bin/
```

## Final Step

Make sure you have logged out of docker by running the following command:

```bash
docker logout
```

Or you can easily remove the docker config file located in ``~/.docker/config.json``.

If you didn't need to remove your docker config file, check its content. It should be empty or like this:

```json
{
  "auths": {},
}
```

Then, simply add ``credsStore`` field to your docker config file and set its value to ``pass``:
```json
{
  "auths": {},
  "credsStore": "pass"
}
```

Now you can run ``docker login`` and enter your docker credentials again. 
If you check the content of your ``~/.docker/config.json`` file, it'll be like this:

```json
{
  "auths": {
    "https://index.docker.io/v1/": {}
  },
  "credsStore": "pass"
}
```

There is nothing related to your credentials stored here. By running the empty ``pass`` command, you'll see that a branch named ``docker-credential-helpers`` is added to your passwords tree like this:

```bash
Password Store
└── docker-credential-helpers
    └── ENCODED_REGISTRY_URL
        └── YOUR_DOCKER_USERNAME
```

That's it. You have now made an effort to make your docker credentials more ``secure``!

Hope this **Zangata** be delicious for you :)