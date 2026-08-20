# An LLM Wiki Changed How I Work

*And everything else I learned about productivity this year*

**By Casey Newton** | Platformer | August 18, 2026 | ~13 min read

**Source:** https://www.platformer.news/karpathy-llm-wiki-journalism-productivity/

---

> *This is a column about AI. My fiancé works at Anthropic. See my full ethics disclosure [here](https://platformer.news/ethics).*

At the beginning of April, the prominent AI researcher Andrej Karpathy [tweeted](https://x.com/karpathy/status/2039805659525644595) out an idea that was, for a certain kind of productivity nerd, an infohazard. He described the idea this way: "Something I'm finding very useful recently: using LLMs to build personal knowledge bases for various topics of research interest." He proceeded to describe his process: adding source documents to a local folder and using an LLM to extract and organize their contents into a Markdown wiki that gets updated as he adds new material.

Karpathy's AI-related pronouncements are closely followed online — he is, after all, the person who [coined](https://x.com/karpathy/status/1886192184808149383) the term "vibe coding." And so, seemingly within hours, the web had filled up with GitHub repos, YouTube videos and Substack posts about how to set up an LLM wiki for yourself. I call the idea an infohazard because, simply by becoming aware of it, I had ensured that I would devote the next several weeks to building it, without having any idea whether it would benefit me at all.

As it so happens, it has. Of everything I tried this year to get better at the deskbound parts of my job, the LLM wiki has easily been the most useful. Like a vintage sports car, it requires a fair degree of maintenance, and there are almost certainly easier ways to create a personal knowledge base. But if you do any sort of work that has made you crave the help of a good research assistant, an LLM wiki might be worth your time.

The wiki is the centerpiece for my annual productivity post, where I run down any changes to the way I work that might be interesting to others afflicted by an inexplicable enthusiasm for software. (Here are the posts from [2023](https://www.platformer.news/why-note-taking-apps-dont-make-us/), [2024](https://www.platformer.news/ai-productivity-apps-capacities-raycast-readwise/), and [2025](https://www.platformer.news/productivity-tools-ai-2025/).)

So with that, here's what I'm still doing from last year, what I stopped doing, and what's new.

---

## What I'm Still Doing

Given how often I switch apps, the best test for whether something actually makes me more productive is longevity. Do I install the app on a new machine? Do I renew the subscription when it's time? Can I point to the places where it actually saves me time?

Three apps I've recommended in previous years still clear that bar.

**[Raycast](https://www.raycast.com/)**, a launcher app that replaces Spotlight, remains my preferred way to navigate a computer. When I press ⌘-space, the Raycast window instantly materializes and lets me perform actions across most of the apps that I use. I look up words; I do math; I reposition windows; I access my clipboard history; I open websites. And thanks to a paid upgrade, I do tons of simple AI searches in the Raycast window. (I use GPT-5.5 Instant here for the high quality-to-speed ratio.) When I'm done, there's no getting lost in a jumble of open tabs or windows — Raycast simply fades back into the background. At this point, I really can't imagine my Mac without it. And in a nice development since last year, it's now available for [Windows](https://www.raycast.com/windows) as well.

**[Capacities](https://capacities.io/)**, which bills itself as "a studio for your mind," is where I keep my daily journal. Each morning, I write a bit about whatever's on my mind. Then, I add notable news links from [Techmeme](https://techmeme.com/) to the bottom of my journal and tag them. The result is that when I'm writing a story or preparing for a podcast, I can click a tag like "Labor" and instantly see all the stories I've saved on that subject since I started building this system a couple years ago. This has been enormously helpful in planning our current podcast miniseries on AI and productivity, since we discuss jobs news on each episode. I used to spend a lot of time digging through databases or running fruitless Google searches in an effort to jog my memory about something I had read; Capacities ensures that it's all just a click away. You could easily do this with any number of apps, but after three years I still find myself appreciating the simplicity and calm of Capacities.

Finally, last year I mentioned testing an app called **[Recall](https://www.recall.it/)** that (anticipating the LLM wiki!) helps you save and organize content from the web. I found myself using it less for that purpose over the past year, except for one remaining killer use case: its Chrome extension provides near-instant text summaries of YouTube videos. Over the past year, I've saved myself many hours by dumping podcasts I feel like I should listen to for work into Recall and simply skimming the summaries.

---

## What I Stopped Doing

The two things I stopped doing over the past year are related to each other — and to the LLM wiki. For years now, I've been seeking a solution to problems of memory. I've been a tech reporter for almost 16 years, have written a newsletter for almost nine, and have written this newsletter for six. For much of that time, I've been publishing stories and saving research materials in various places. And when a news story comes along that draws on some of that history, I want to find that context as quickly as possible.

Last year I talked about how Notion had shipped a feature I'd wanted for years: an agent that could search across the thousands of links I had saved into it over the years. It worked well enough, but I couldn't turn it into a habit. The search feature in the database itself is a simple keyword-based search; agentic search takes place on one of the app's many other surfaces; and the agent often failed to cite its sources without additional prompting. It worked, but it was effortful.

Moreover, it only looked backward. I also had a need for a system that would help me organize stories around new concepts, and at scale. In Capacities, I took to creating pages I called "blips" (inspired by Andy Matuschak) where I could gather loose string. For example, in the summer of 2024 I created a blip called "AI could create massive job loss," and added relevant links to stories about AI and jobs as I encountered them during my daily journaling.

For a few months I felt extremely clever, because I created a dynamic object in the template for my daily journal in Capacities that would show me a selection of blips at random. This would regularly remind me of blips that I had forgotten to update, and worked better than any system I had previously devised for tracking long-term stories.

Ultimately, though, even this system asked a bit too much of me. As the number of blips proliferated, my ability to consistently track stories across all of them waned. This, in turn, made me more reluctant to create new blips.

And that's why, when I saw Karpathy's tweet, I sat a little straighter in my chair. **What if AI could write and update all those blips for me?**

---

## The LLM Wiki

Really, the most important change in my productivity over the past year is that… I make software now? Like seemingly everyone else at the end of last year, I began messing around with Claude Code, and have since made a small handful of apps that I really do use all the time. Sometimes I use [Glaze](https://www.glaze.app/), another app from the maker of Raycast; it excels at design and polish.

For the LLM wiki, though, I just asked Claude Fable 5 to write me a prompt that would get me a "Karpathy-style LLM wiki." I pasted the result into the terminal (I use [Ghostty](https://ghostty.org/)), and before too long I had created a new folder of Markdown files in Obsidian.

One thing that makes the wiki particularly valuable for me is that I first **seeded it with my own writing**: the entire Platformer archive, from which Claude expertly extracted all the various people, companies, and concepts that I've covered here since 2020 and wrote them up in Markdown files that live on my computer.

These files can get quite long; my page for Meta runs to more than 12,000 words and contains more than 1,300 links to other pages in the wiki. Day to day, that isn't of much practical use. But I've lost more than one afternoon browsing the archive in the same state of blissed-out curiosity that I browse Wikipedia, remembering old stories and reasoning about how they fit into current events.

But I wanted more than a wiki of my own work. And so now **each morning after I journal, I save a selection of stories into the wiki** via Obsidian's web clipper, which converts them into Markdown. Then a script on my computer reads the stories and figures out where they fit into the wiki.

The result is that I now have more than **1,440 wiki pages** covering most of what has ever interested me at Platformer: from the content moderation focus of the first few years to my growing interest in child safety and AI progress. The wiki creates detailed timelines that link to original sources, and I can ask it questions using a simple Obsidian plugin named [Claudian](https://community.obsidian.md/plugins/realclaudian).

### How it makes me more productive

I've found it highly useful in fast-evolving, complex cases — like the OpenAI/Hugging Face agentic breach, where we learned a little more about the story every few days for a matter of weeks. Each day as I prepared to write, or podcast, or go on someone else's podcast, I would pull up the page and refresh my memory — while also opening up the original sources to make sure nothing I was about to say was hallucinated. Given the real complexity of that story, this saved me tons of time.

It also gives me **useful story ideas** — and it does it by automating my old blips system. Each day as it reads, the wiki generates new pages for concepts in the news — like "tokenmaxxing," or "youth social media bans," or "AI copyright." It also updates a "home" page every morning that highlights stories in the news. This week, spotting the "AI and Congress" concept in my wiki led me to open the page and see a number of recent stories on the subject; I later pitched it as a podcast segment.

### The downsides

All of that is great. What's less great is that the wiki **needs more or less constant maintenance**. Pages grow too long and need to be compacted. An error in the code means that one process or another stops running and has to be fixed. And Claude's hyper-compressed, borderline-unreadable house style led me to use GPT-5.6 Sol to rewrite much of the system to more closely approximate AP style. (It did a great job.)

It has now been just over a month since I created the wiki, and I find myself looking forward to checking in on it in the morning to see what it has built. On one hand, it seems too specific to me and too clunky for me to confidently recommend. On the other, "self-organizing knowledge base" strikes me as the **teleological end of whatever process began the day I first installed Evernote** on my Mac in 2008.

That's one reason why I sought out Town CEO Jean-Denis Greze for an interview last week; one way of thinking about that product (or what it might evolve into) is a kind of Karpathy LLM wiki for work. As ever, the price of using these tools at the bleeding edge is that I arrive everywhere much too early. The payoff is that, somehow, I am enjoying myself quite a lot.

---

## Key Takeaways

- **The core idea (Karpathy's):** Drop source documents into a folder → LLM extracts/organizes them into a Markdown wiki → wiki auto-updates as you add material
- **Casey's setup:** Claude Fable 5 generated the prompt/script → Markdown files in Obsidian → Obsidian web clipper for daily ingestion → Claudian plugin for Q&A
- **Seeding trick:** Bootstrapped with his entire newsletter archive (6 years of Platformer), giving it deep institutional knowledge
- **Daily workflow:** Morning journaling → save select stories into wiki via web clipper → script updates relevant wiki pages
- **Scale achieved:** 1,440+ pages in ~1 month
- **Best use case:** Fast-moving, complex stories where you need to rapidly re-orient to accumulated context
- **Story idea generation:** Wiki auto-creates pages for emerging concepts; home page surfaces what's trending in your beat
- **Tradeoffs:** Requires ongoing maintenance; Claude's default writing style needed to be overridden; code breaks and needs fixing

---

*Tags: #productivity #LLM #knowledge-management #Obsidian #journalism #PKM #AI-tools*
