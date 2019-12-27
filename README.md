# google-auth-2FA
2FA-ubuntu


Sometimes, you want to be doubly sure that no one can get access to your VPS but you. If two-factor authentication (2FA) can work for a dashboard, why not for a VPS? In this tutorial, we’ll talk about enabling SSH two-factor authentication (2FA) on an Ubuntu 16.04 VPS, via Google Authenticator (or another 2FA app of your choice) so that you can put another layer of protection on your VPS.

It’s important to note that the most common and recommended method of securing SSH-based logins is SSH keys, and we heartily recommend you use them on all your VPSs.

For the sake of explaining how SSH two-factor authentication works, we’ll start with using 2FA to protect an SSH configuration that uses passwords, not SSH keys. We’ll follow that up with a recommendation on how you might use both SSH two-factor authentication and SSH keys for a rock-solid (and perhaps overkill) triple-factor SSH authentication.

Updated on May 25, 2018!
## Prerequisites

A VPS running Ubuntu 16.04
A non-root, sudo-enabled user

## Steps

### Step 1. 

Installing Google Authenticator on the VPS

First off, you need to install the Google Authenticator program and run it as the user you’re going to log in with—preferably not the root account.

```
sudo apt-get install libpam-google-authenticator
```

```
google-authenticator
```

You’ll first be prompted as to whether or not you want to use time-based tokens. The other option is one-time tokens, which is probably not what you want. Type y to use time-based tokens.

Do you want authentication tokens to be time-based (y/n) y

Once you confirm time-based tokens, you’ll see a large amount of output on your terminal. This will include the QR code that you can use with Google Authenticator (or another similar program), or a secret key that you can enter in manually if your device can’t scan QR codes.

Once you’ve set up the token on your mobile device, you can take a look at the secret keys that the program generates, along with emergency scratch codes, which you can use once if you lose that device.

There are a few more questions to answer—the first of which is whether you want to save the Authenticator settings. Answer in the affirmative.

Do you want me to update your "/home/USER/.google_authenticator" file (y/n) y

The next option helps reduce the risk of certain attacks, so you’ll want to use it as well.

Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases

your chances to notice or even prevent man-in-the-middle attacks (y/n) y

The next option, to help reduce the effects of time-skew, probably isn’t necessary. If you do have problems later on, you can re-run this configuration and select y instead.

By default, tokens are good for 30 seconds and in order to compensate for
possible time-skew between the client and the server, we allow an extra
token before and after the current time. If you experience problems with poor
time synchronization, you can increase the window from its default

size of 1:30min to about 4min. Do you want to do so (y/n) n

The last option allows you to set a limit on how many times someone can attempt to use security tokens—no more than 3 times every thirty seconds. This dramatically reduces the risk of brute force attacks.

If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.

Do you want to enable rate-limiting (y/n) y

### Step 2. 

Copy the code to your smartphone

Scroll up in your terminal, either with the mousewheel or Page Up, and take a look at the QR code, secret key, verification code, and emergency scratch codes. You might want to copy the text into a document for temporary safekeeping, just in case you lose the terminal output during this process.

To add the code to your smartphone, open the Google Authenticator app and tap Scan a barcode. Give the app permission to take photos if it asks. Then, simply point your phone at the QR code in the terminal. You should hear a small beep, and you’ll then start to see 6-digit codes that change every so often. You’ll use those codes when logging in via SSH, so let’s turn back to your server.

Step 3. Enable Google Authenticator in SSH
Before we go any further, it’s best to work in multiple SSH connections in multiple terminals beyond this point. If you try this using only a single terminal and SSH connection, you might accidentally lock yourself out of your VPS due to misconfiguration in your SSH settings. By keeping one SSH connection active and using a second terminal to test these new settings, you can always make further tweaks or roll back to a setting that works.

Note: In order for this particular configuration to work, PubkeyAuthentication must be set to no and PasswordAuthentication must be set to yes. More on using 2FA plus SSH keys in the next step!

In order to enable these codes via SSH, edit the /etc/pam.d/sshd file and add the following line at the beginning.

auth required pam_google_authenticator.so
You then need to open /etc/ssh/sshd_config and look for the ChallengeResponseAuthentication line. If you have one, you can simply change no to yes, and uncomment it (remove the #) if necessary. If you don’t have this line, add the text below.

Either, way the line in question should look like this:

ChallengeResponseAuthentication yes

Finally, you just need to restart sshd.

```
sudo systemctl restart sshd
```

At this point, you can try connecting to your VPS via another terminal. You’ll first be asked for your password, followed by the verification code. If all is working well, you’ll be logged in!

### Step 4 (optional). 

Reconfigure SSH to use both 2FA and SSH keys

### Step 3 

relied upon disabled SSH key authentication, and forced password authentication. SSH keys are a reliable way to improve SSH security, whereas 2FA is a bit of an unknown. So, let’s figure out how to use both SSH keys and 2FA for maximum factors.

First, hop back into your /etc/pam.d/sshd file and comment out the auth required pam_google_authenticator.so we added earlier. It should look like this:

### auth required pam_google_authenticator.so

Now, above that line, add another:

auth [success=done new_authtok_reqd=done default=die] pam_google_authenticator.so nullok

In your /etc/ssh/sshd_config file, ensure that PubkeyAuthentication is set to yes and is PasswordAuthentication to no.

In this same file, you also need to change the authentication methods that are used for SSH logins:

AuthenticationMethods publickey,keyboard-interactive
Finally, restart sshd again.

```
sudo systemctl restart sshd
```

Now, when you log into your VPS, you are asked first for the passphrase for your SSH key, followed by your 2FA code. Pretty cool!


https://www.howtogeek.com/121650/how-to-secure-ssh-with-google-authenticators-two-factor-authentication/
