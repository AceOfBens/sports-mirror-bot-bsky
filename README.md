# Build your own Bluesky mirror bot 🦋

This is a template repo for building a [Bluesky](https://bsky.app/) bot that mirrors an X/Twitter account by pulling from existing [sports mirror bots on Mastodon](sportsbots.xyz) as a work around for lack of access Twitter's API.

## Table of Contents
* Credits
* Instructions
* FAQ

## Credits
* [Phil Nash](https://github.com/philnash) for providing the code for building a bot that posts on its own schedule. [Phil Nash's Bluesky bot template](https://github.com/philnash/bsky-bot)
* [acarters](https://github.com/acarters) for providing the code for building a bot that mirrors a X/Twitter account by pulling from existing mirror bots on Mastodon, due to the access restrictions on X's API.
* I ([Ben Ace](https://bsky.app/profile/aceofbens.com/)) cannot stress enough how much I didn't do much to this code and can't take credit for any the building of this repo, but I did edit this template and outline the instructions below! I am not a developer but a [graphic designer](https://aceofbens.com/) whose coding experience ends at tinkering with HTML and CSS on WordPress once in a while.

## Instructions

Below, I outline the places in each file where you will need to add something specific to the code to connect it to the mirror account you have set up on Bluesky and pull from the correct Mastodon account, as well as a few features that do not need to be changed to operate but are helpful to know about in case you want to change the settings. Not all file need to edited, and, in fact, most of them should not be touched unless you are a developer who knows what you are doing!

### [/ .github / workflow / post.yml](.github/workflow/post.yml)

#### $${\color{#81b812}Line 4}$$
`- cron: "* * * */12 *`

**Purpose:** This line tells the bot how often to run.

**What to change:** This format can be confusing at first, but an easy way to help you visualize it is to use [crontab guru](https://crontab.guru/). Typically, I keep this setting at every 20 minutes, which would be `*/20 * * * *` by this format.

Whatever you set is what Github will use as a guideline, but from experience, I will tell you that no matter what you set, Github will run on its own timeline, meaning that if you schedule it to run every 20 minutes, then during the day (Eastern Time), it will run every 10 to 30 minutes, and on many evenings it will run every 30 minutes to 2 hours.

If the bot is slow to catch up, you can go to the Actions tab, select the most recent workflow (Post to Bluesky), and re-run all jobs to push the bot to run again, if you so wish.

#### Lines 14-15
```
- run: npm install axios
- run: npm install tsl-mastodon-api
```

**Purpose:** These must be installed for the bot to run properly.

**What to change:** After the bot runs the first time, these two lines can be removed as they do not need to be installed on every workflow.

#### Lines 21-22
```
BSKY_HANDLE: ${{ secrets.BSKY_HANDLE }}
BSKY_PASSWORD: ${{ secrets.BSKY_PASSWORD }}
```

**Purpose:** This gives the bot the login info for the Bluesky account you've set up for it.

**What to change:** DO NOT edit these lines directly!! Instead of adding the username and password directly to the code, go to the repository's Settings > Secrets and variables (under the Security heading) > Actions > New repository secret

For **Name**, type in `BSKY_HANDLE` or `BSKY_PASSWORD`
For **Secret**, type in the handle (ex. @notphillies.bsky.social) or the password (can be the password you set up on Bluesky or an app password).

### [/ src / lib / bot.ts](src/lib/bot.ts)

#### Line 29
`dryRun: true,`

**Purpose:** Toggle whether the bot is in test mode.

**What to change:** For the sake of organization, I included this step here, but **this should be done last** upon set up. When you are ready for the bot to begin running, change `true` to `false`.

This can be toggled on and off (`false` or `true`, respectively) for any reason in the future, such as if you are editing the code and want to avoid the possibility of the bot automatically running before you've fixed the bug or changed whatever settings you're trying to update.

#### Line 83 (optional/if needed)
`var postNum = 20;`

**Purpose:** Tells the bot how many posts to check on the Mastodon account when checking for updates.

**What <ins>can</ins> change:** I (Ben Ace) have temporarily lowered this number to 5 (or even lower) when I've decided to update other settings in a way that would cause the bot to double post. 

For example, I add settings for the bot to substitute players' X/Twitter handles with their first and last name in the Bluesky post. When a new player joins the team, and the team account tags them, I will add them to the list of substitutions. If the Bluesky post has already received engagement (quotes, reposts, replies), typically, I will leave it up, and if it is in the 20 most recent posts from the Mastodon mirror bot, this Bluesky bot will double post with the newly added substitution.

To work around this, I change this setting to a lower number (ex. if the post I don't want it to double post is the third most recent, I will change this setting to 2). Then, the next day, I will change it back to 20. Keep in mind that if the account you're mirroring posts frequently, setting this number too low will potentially result in missed posts.

#### Line 84
`var bskyFeedAwait = await this.userAgent.app.bsky.feed.getAuthorFeed({actor: "CHANGETHIS.bsky.social", limit: postNum,});`

**Purpose:** Tells the bot which Bluesky account's feed to compare the posts from the specified Mastodon account to.

**What to change:** Put the Bluesky handle of the account you have set up for your mirror bot (ex. notphillies.bsky.social) in place of `CHANGETHIS.bsky.social`.

#### Line 149 (Change if/when Bluesky updates their video length limit from 3 minutes)
`if (urls[0].slice(-3) == "mp4" && parseFloat(alts[0].split("@#*")[2]) < 180 && limits.canUpload == true)`

**Purpose:** Gonna be so real with y'all, I (Ben Ace) am not entirely sure what the purpose of the line is, but it has something to do with the time limits on Bluesky videos.

**What to change:** When Bluesky updated their limits on video from 1 minute to 3 minutes, I noticed that the bots were still pulling the thumbnail of videos longer than a minute and adding a note in the alt text that the video exceeds Bluesky's limits. After digging around the code for a bit, I noticed that where it currently says `180` in the line above, it had said `60`. So, if Bluesky increases the time limit on videos again, update this line with the new time limit in number of seconds.

#### Line 175
`cardResponse = await axios.get("https://a.vsstatic.com/mobile/app/mlb/logos/app/philadelphia-phillies-app-2.jpg", { responseType: 'arraybuffer'});`

**Purpose:** If the bot has trouble pulling a card from the source post and buffers for too long, this is the image it will post in place of the card.

**What to change:** Currently, there is an image file with the Flyers logo on it in this setting. I have yet to see the bot post this photo, but just in case your bot does, change this photo to avoid embarrassment.

#### Line 218 (optional/if needed)
`var postNum = 20`

**Purpose:** Tells the bot how many posts to check on the Bluesky account when checking for updates.

**What <ins>can</ins> change:** Same as for Line 83, this can be edited if you're trying something out. Personally, I haven't found a need to change Line 218 yet, but I wanted to point it out in case it's useful.

### [/ src / lib / getPostText.ts](src/lib/getPostText.ts)

#### Line 2
``

**Purpose:**

**What to change:**

#### 
``

**Purpose:**

**What to change:**

#### 
``

**Purpose:**

**What to change:**

### No Change Needed

The following files in this bot **do not** need to be edited to run properly, and unless you are well versed in Typescript, it is highly recommended that you do not change anything in them.

* / src / index.ts
* / src / lib / config.ts
* / .env.example
* / .gitignore
* / .nvmrc
* / LICENSE
  * Note: The license details from Phil Nash's Bluesky bot repository is included in this template because his work is the basis for this repository.
* / README.md
  * Note: As this is a text file meant to give you the necessary information to set up your Bluesky mirror bot, technically you can change anything in here and the bot will still run properly. That said, if you make your repository public or make it private intend to link to it anywhere, **do not** remove the credits section at the top.
* / eslint.config.js
* / package-lock.json
* / package.json
* / tsconfig.json
* / yarn.lock


## FAQ

### Does this cost anything?

Generally, no. The purpose of pulling from a Mastodon mirror bot instead of directly from the X/Twitter account is because accessing X/Twitter's API is a hassle for several reasons. So, that part of it costs nothing. However, depending on how many bots you have running through your Github account and how often you schedule them to run, you may hit the monthly limit

### I've noticed that [@notflyers.bsky.social](https://bsky.app/profile/notflyers.bsky.social/) often links to a YouTube video from the Philadelphia Flyers' channel. Why isn't my bot doing that?

Whenever [@notflyers.bsky.social]((https://bsky.app/profile/notflyers.bsky.social/)) or [@notphillies.bsky.social](https://bsky.app/profile/notphillies.bsky.social/) has a YouTube video embedded, I (Ben Ace) posted that manually. I'm not sure how to set up the bot to search a YouTube channel for a corresponding video when the video they've posted to X/Twitter exceeds Bluesky's 3-minute video limit, so, if I'm available before the bot has caught up, I will post manually with the YouTube embed. If I catch it after the bot has already posted it, I'll typically link to the corresponding YouTube video in a reply.

Yes, I have too much time on my hands. It's called un(der)employment.

### I've noticed that [@notphillies.bsky.social](https://bsky.app/profile/notphillies.bsky.social/)'s profile picture and banner update for Thursday and Friday home games like it does on X/Twitter. Why isn't my bot's profile picture or banner updating when the X/Twitter account it's mirroring updates them?

That's another thing I (Ben Ace) do manually. Again, I know I have too much time on my hands.

#### Credits

[Phil Nash](https://github.com/philnash/bsky-bot): 

This is a template repo for building [Bluesky](https://bsky.app/) bots that post on their own schedule. It uses [TypeScript](https://www.typescriptlang.org/) to build the bot and [GitHub Actions](https://docs.github.com/en/actions) to schedule the posts.

* [How to use](#how-to-use)
  * [Things you will need](#things-you-will-need)
    * [A Bluesky account](#a-bluesky-account)
    * [Node.js](#nodejs)
  * [Create a new repository from this template](#create-a-new-repository-from-this-template)
  * [Running locally to test](#running-locally-to-test)
  * [Create your own posts](#create-your-own-posts)
  * [Deploy](#deploy)
    * [Schedule](#schedule)
    * [Environment variables](#environment-variables)
  * [Set it live](#set-it-live)


## How to use

### Things you will need

#### A Bluesky account

To use this repo you will need a [Bluesky account](https://bsky.app/). [Sign up for an invite here](https://bsky.app/).

Once you have an account for your bot, you will need to know your bot's handle and password (I recommend using an App Password, which you can create under your account's settings).

#### Node.js

To run this bot locally on your own machine you will need [Node.js](https://nodejs.org/en) version 18.16.0.

### Create a new repository from this template

Create your own project by clicking "Use this template" on GitHub and then "Create a new repository". Select an owner and give your new repository a name and an optional description. Then click "Create repository from template".

Clone your new repository to your own machine.

```sh
git clone git@github.com:${YOUR_USERNAME}/${YOUR_REPO_NAME}.git
cd ${YOUR_REPO_NAME}
```

### Running locally to test

To run the bot locally you will need to install the dependencies:

```sh
npm install
```

Copy the `.env.example` file to `.env`.

```sh
cp .env.example .env
```

Fill in `.env` with your Bluesky handle and password.

Build the project with:

```sh
npm run build
```

You can now run the bot locally with the command:

```sh
npm run dev
```

This will use your credentials to connect to Bluesky, but it *won't actually create a post yet*. If your credentials are correct, you should see the following printed to your terminal:

```
[TIMESTAMP] Posted: "Hello from the Bluesky API"
```

To have the bot create a post to your Bluesky account, in `index.ts` change line 4 to remove the `{ dryRun: true }` object:

```diff
- const text = await Bot.run(getPostText, { dryRun: true });
+ const text = await Bot.run(getPostText);
```

Build the project again, then run the command to create a post to actually create the post with the API:

```sh
npm run build
npm run dev
```

### Create your own posts

Currently the bot calls on the function [`getPostText`](./src/lib/getPostText.ts) to get the text that it should post. This function returns the text "Hello from the Bluesky API" every time.

To create your own posts you need to provide your own implementation of `getPostText`. You can do anything you want to generate posts, the `getPostText` function just needs to return a string or a Promise that resolves to a string.

### Deploy

Once you have built your bot, the only thing left to do is to choose the schedule and set up the environment variables in GitHub Actions.

#### Schedule

The schedule is controlled by the GitHub Actions workflow in [./.github/workflows/post.yml](./.github/workflows/post.yml). The [schedule trigger](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule) uses cron syntax to schedule when the workflow runs and your bot posts. [Crontab Guru](https://crontab.guru/) is a good way to visualise it.

For example, the following YAML will schedule your bot to post at 5:30 and 17:30 every day.

```yml
on:
  schedule:
    - cron: "30 5,17 * * *"
```

Be warned that many GitHub Actions jobs are scheduled to happen on the hour, so that is a busy time and may see your workflow run later than expected or be dropped entirely.

#### Environment variables

In your repo's settings, under *Secrets and variables* > *Actions* you need to enter two Secrets to match your `.env` file. One secret should be called `BSKY_HANDLE` and contain your Bluesky username, and the other should be called `BSKY_PASSWORD` and contain your App Password that you generated for the bot account.

### Set it live

Once the schedule is set up and your Environment variables configured, push your changes to your repo and wait for the schedule to trigger the workflow. Your bot will start publishing posts based on your code.

If you have any issues with that, please [raise an issue in this repo](https://github.com/philnash/bsky-bot/issues) or send me a message on Bluesky [@philna.sh](https://staging.bsky.app/profile/philna.sh).
