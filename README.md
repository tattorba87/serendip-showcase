<div align="center">
  <img src="media/serendip-icon-purple-coral-512.png" alt="" width="96" height="96">
  <h1>Serendip</h1>
  <p><b>Discover things to do in your city, and actually do them.</b></p>
</div>

Most people end up at the same few places. The options are out there, but finding them is work:
Eventbrite, Facebook events, a handful of local listings sites, none of them agreeing with each
other. Serendip collects what is on in a city and puts it on a map you can act on in a few taps.

I built all of it: the phone app, the API, the database, and a nightly pipeline that reads event
pages with an LLM and turns them into structured rows.

Montréal, on a real phone, against real data.

## Demo

<div align="center">
  <!-- Wrapped in its own anchor on purpose: GitHub linkifies bare markdown images to the
       file's blob page, which navigates away from the README. Its own <a> suppresses that,
       and pointing it at this section's heading keeps a click on the page. -->
  <a href="#demo"><img src="media/demo.gif" alt="Serendip demo" width="400"></a>
</div>

<p align="center"><i>One take, real time, on a Samsung A05s. Browsing suggestions on the map,
saving one, filtering to free events, then asking the Search tab a question in plain English.<br>
<a href="media/demo.mp4">Full quality recording</a></i></p>

## A closer look

| Map | Event | Search |
|---|---|---|
| <img src="media/screens/01-map.png" alt="Map of Montréal with clustered event pins"> | <img src="media/screens/02-detail.png" alt="Event detail sheet"> | <img src="media/screens/03-search.png" alt="Conversational search answering with event cards"> |
| Everything on in the city, clustered | Details, with a one line summary an LLM wrote from the source page | Ask in plain English. The answers are real events, with suggested follow ups |

## How it's built

![How Serendip is wired together](media/diagrams/big-picture.png)

Two loops that stay out of each other's way. At 2am a Python pipeline pulls from about a dozen city
sources, uses an LLM to turn messy event pages into structured rows, drops the duplicates that show
up when two sources list the same thing, and works out each user's feed in advance. When you open
the app, the API only reads what is already sitting in Postgres, so the map and the feed cost no
model call at all. The Search tab is the one place a model runs while you wait.

The models run on a GPU box in my apartment, which also hosts the API and the nightly job. That was
a deliberate choice while prototyping: running inference on my own hardware costs nothing per call,
so I could rewrite prompts, rebuild every embedding in the corpus and rerun the whole pipeline as
often as I wanted without watching a bill. The only recurring costs were a domain name and an Apple
developer account. Every model call goes through a provider layer, so moving to a hosted API is a
config change rather than a rewrite.

The phone reaches the box over a tunnel that only makes outbound connections, so nothing is exposed
at home. Releases go out through TestFlight and Google Play internal testing.
