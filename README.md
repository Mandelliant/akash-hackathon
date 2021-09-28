# akash-hackathon
---

To run Discourse on Akash with a decentralized domain and mailserver, you'll need:

* Akash & a funded wallet
* [Docker](https://medium.com/r/?url=https%3A%2F%2Fwww.docker.com%2Fproducts%2Fdocker-desktop) + a [Docker Hub account](https://medium.com/r/?url=https%3A%2F%2Fhub.docker.com%2Fsignup)
* A Handshake domain and the ability to set records


<h1>Step 1: Run Akash</h1>

I'm using Windows, for Mac or Linux installation steps, see the [official documentation](https://docs.akash.network/guides/command).

1. Install [golang](https://medium.com/r/?url=https%3A%2F%2Fgolang.org%2F) version 1.16 or higher
2. Run the GoInstaller command:
`> curl -sSfL https://raw.githubusercontent.com/ovrclk/akash/master/godownloader.sh | sh`
3. Open the folder, extract the binaries, and from the command line, navigate to the folder with the Akash code. Run: `> akash`
4. Create an account and name the wallet: `> set AKASH_KEY_NAME=myWallet`
5. Run `> akash keys add %AKASH_KEY_NAME%`
6. Create an account address variable: `> akash export AKASH_ACCOUNT_ADDRESS="%(akash keys show %AKASH_KEY_NAME% -a)"`
7. Set a keyring backend variable: `> set AKASH_KEYRING_BACKEND=os`
8. Set your Base URL: `> set AKASH_NET="https://raw.githubusercontent.com/ovrclk/net/master/mainnet"`
9. ... version: `> set AKASH_VERSION="%(curl -s "%AKASH_NET%/version.txt")%"`
10. ... ChainID: `> (for /F %I in ('curl -s %AKASH_NET%/chain-id.txt') do IF NOT DEFINED AKASH_CHAIN_ID (set AKASH_CHAIN_ID=%I)) > NUL`
11. ... Network node: `> (for /F %I in ('curl -s %AKASH_NET%/rpc-nodes.txt') do IF NOT DEFINED AKASH_NODE (set AKASH_NODE=%I)) > NUL`

Confirm all of the variables are setup by running `echo %AKASH_NODE% %AKASH_CHAIN_ID% %AKASH_KEYRING_BACKEND%`.
The result should look similar to `http://135.181.60.250:26657 akashnet-2 os`.

12. Fund your wallet with AKT from an [exchange](https://medium.com/r/?url=https%3A%2F%2Fakash.network%2Ftoken), a [developer grant](https://medium.com/r/?url=https%3A%2F%2Fakash.network%2Ftoken), or a DEX like [Osmosis](https://medium.com/r/?url=http%3A%2F%2Fosmosis.zone).

You can check that your AKT are in your account by running `akash query bank balances --node %AKASH_NODE% %AKASH_ACCOUNT_ADDRESS%`

13. Download an example deployment file: `curl -s https://raw.githubusercontent.com/ovrclk/docs/master/guides/deploy/deploy.yml > deploy.yaml`
14. Create your authentication certificate: `akash tx cert create client --chain-id %AKASH_CHAIN_ID% --keyring-backend os --from %AKASH_KEY_NAME% --node %AKASH_NODE% --fees 5000uakt`


<h1>Step 2: Configure mail server</h1>

You can use a Handshake domain with hmail, a Handshake email client, to set up a basic Discourse instance. Discourse also [recommends a few options](https://medium.com/r/?url=https%3A%2F%2Fgithub.com%2Fdiscourse%2Fdiscourse%2Fblob%2Fmain%2Fdocs%2FINSTALL-email.md) or you can set up your own mail server. For Discourse mail deliverability, you **must** be able to add valid SPF and DKIM records in your DNS.

Once you have a mail server set up, get the following information and save it for later when installing Discourse:

```
SMTP server address? [smtp.example.com]: 
SMTP port? [587]: 
SMTP user name? [user@example.com]: 
SMTP password? [pa$$word]:
```

<h1>Step 3: Get a Handshake domain</h1>

You can get a Handshake domain though [Namebase](namebase.io) or buy Handshake domains on [Namecheap](namecheap.com).

Use the method of your choice to get a name. After Discourse is installed, you need to return to your domain manager to configure the domain.


<h1>Step 4: Install Discourse</h1>

Open a new command line tool window by right-clicking and selecting Run as administrator, then clone the official Discourse Docker image into `/var/discourse`: `> git clone https://github.com/discourse/discourse_docker.git /var/discourse
cd /var/discourse`


**Note:** you will need to be root through the rest of the setup and bootstrap process so run Discourse related commands in your CLI window with admin privileges.


<h1>Step 5: Create your Akash deployment</h1>

With Docker Desktop open, navigate into the newly downloaded Discourse image and open the Dockerfile in `/var/discourse/image/base`.

Run `docker build --tag yourName .` to build the Discourse Docker image from the Dockerfile. Alternatively, you can download the image from [Docker Hub](https://medium.com/r/?url=https%3A%2F%2Fhub.docker.com%2Fr%2Fdiscourse%2Fbase%2Ftags%3Fpage%3D1%26ordering%3Dlast_updated). If you're using Docker Desktop, make sure the image is in use by clicking `Run`.

<h2>Modifying the deploy.yaml file</h2>

I'm still working through this part.

Need to use Akash SDL to determine what specifications to include in the `deploy.yaml` file when deploying to Akash. Discourse needs to be bootstrapped once deployed to a cloud instance, including answering user prompts to configure the download. For setup, Discourse also requires you to create a DNS A record for your server's hostname in your DNS manager pointing to the IP address of the cloud instance where you're installing Discourse (which can't be obtained until you've successfully deployed to the Akash network).

**Next steps**
* Set Discourse config values as env variables in `deploy.yaml`
* Set commands in `deploy.yaml` to map variables to corresponding Discourse config prompts
* Test deployment until manifest successfully uploaded to provider and can deploy
* Get provider IP address & add A record to DNS settings for the domain

Once the `deploy.yaml` file is correct and I can successfully complete bootstrapping, the Discourse server should be accessible through a web browser via the Handshake domain name used during configuration.
