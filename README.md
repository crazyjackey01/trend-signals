# Trend Signals

Trend Signals shows what people are searching for right now and why. Every sentence links to the article or number it came from.

Live site: https://trend-signals.gimailft01.workers.dev

Anyone can read past results. A new search needs the password in my LMS submission.

## Purpose and intended user

This is for anyone who wants to know what people are searching for a lot right now, why they are searching for it, and what might happen next. I didn't want another AI that gives a confident answer with nothing behind it, so code collects and measures the data first. The language model can only write from those numbered facts, and it has to cite them.

The next step, which I haven't started, is getting Google's AI search to cite this project's data, then checking whether it does.

## How to run it

On the website, type a topic, or leave the box empty to see what's trending. Hover over part of an analysis to see how it was made. Past analyses and test results are in the History tab.

The local version (better because Google News refuses Cloudflare) also has article counts and forecasts. Needs Python 3:

```sh
python -m venv .venv && source .venv/bin/activate
pip install -r midterm/requirements.txt
# make a .env file with OPENROUTER_API_KEY=... and OPENCODE_API_KEY=...
python midterm/app.py        # opens localhost:8765
```

A search takes about a minute and costs about $0.03. Test details: `midterm/eval/README.md`.

## How it works

1. Collect Google autocomplete phrases and news, and remove gambling spam (too much slop).
2. Using plain code, keep the headlines that name the topic, then work out search purposes, suddenly common words and dates.
3. Using plain code, forecast next week's Google search interest from 5 years of data.
4. Claude Sonnet writes the explanation from the numbered facts. Any sentence without a valid citation gets deleted.
5. Jev, a model that can only pick from labels I give it, rates each sentence against its sources. Sentences rated "unsupported" get removed.

## Work process

I came up with the idea and started building in October. I built the collector and cited analysis, then the web page, then a public version on Cloudflare. I made the decisions about what to build and what to keep, and Claude Code wrote most of the code.

Then I tested it. Some of my first checks didn't prove much, so I removed them. I also moved search-purpose sorting, article filtering and forecasting from models to plain code, and redid the Jev test so it was blind.

## Selected Claude Code evidence

My prompts as I typed them.

**1. Oct 11.** Claude had a trending-search collector working and wanted to move on.

> ok wait. are search things from yesterday the best you can do? and also, i want to tweak the search finding thing to find things related to what the searcher wants.

This is why you can search any topic.

**2. Oct 11.** Claude told me to sign up for Naver and GNews keys.

> no like we should be able to do this on google

The public site gets its news through Gemini with Google Search instead.

**3. Oct 11.** Claude suggested starting with Gemini 3.6.

> i think just looking for search things dont need 3.8 no?

After testing, the lightest model, Flash-Lite, goes first.

**4. Oct 11.** I wanted to know if Jev was worth using.

> the purpose of jev is that it dosent have hallucinations and can conserve tokens. is it working?

Sonnet turned out to be the main cost, which Jev couldn't reduce. I chose to add an article filter.

**5.** I labelled 30 sentences by hand without seeing Jev's answers, then scored them:

```
30 labelled claims: agreement 53%, Cohen's kappa -0.09
```

**6. Nov 1.** That score was bad, so I asked:

> if the kappa score is that bad, can you make it so that if actualy improves over time, with data, or labling?

Labels can now be shown to Jev as examples, but only if a held-out test shows they help. They didn't, so accuracy hasn't improved:

```
25 labels: kappa -0.062 with examples, -0.02 without -> examples off
```

## Verification results

Run on November 1. No model grades its own output.

| Check                                           | Result                                                                |
| ----------------------------------------------- | --------------------------------------------------------------------- |
| Normal search: 스마트 안경                      | Worked. 189 articles kept, 164 dropped, 8 cited sentences             |
| Edge case: made-up topic `zzqxv 프로젝트`     | No crash. 0 articles, no forecast, 2 sentences saying there's no news |
| Google search forecast, last 52 weeks, 4 topics | Beat "same as last week" on 0 of 4                                    |
| Real values inside the 80% forecast range       | 72%, across Google and Wikipedia                                      |
| Jev vs my 30 blind labels                       | Kappa -0.09, no better than chance                                    |
| Jev rating the same sentences twice             | Same label on 29 of 30                                                |

The 스마트 안경 run also had a real failure. Sonnet wrote that exam-related phrases were the majority of autocomplete suggestions, but the source only says 54% were "other", with exam phrases as examples. Jev marked it "partly" and it stayed on the page.

## Three practices that helped

1. I tested ideas before switching them on. The label examples failed before they could affect any results.
2. I committed small changes and kept self-checks in the code, so mistakes showed up early.
3. I had Claude tell me what it hadn't checked. It once claimed Google blocks Cloudflare from forecast data when it had only tested Google News.

## Three difficulties

1. Jev and I disagree. It never rejected a sentence in the test, including 4 I marked unsupported, and showing it my labels didn't help. This isn't solved. I stopped at 30 labels from one person, so I can't say which of us is off.
2. Google News refuses Cloudflare. I switched the public site to Gemini search, which works, but that version still has no article counts or forecasts.
3. The forecasts don't beat a simple guess. No method beat "same as last week" on Google data. The page says this and shows a range.

## Known limitations

- It only reads headlines and summaries, not full articles.
- The "matches its sources" count isn't validated, because Jev failed its test.
- On new topics the word lists catch only 32-48% of search phrases. The rest show as "other".
- The headline filter drops articles that use a different name for the topic.
- Forecast ranges can be too wide to be useful (-100% to +85% for 스마트 안경), and new terms get no forecast.
- Google News caps queries at 100 articles, so busy days get undercounted.
- "What happens next" is only a forecast of next week's search interest, not of events, and it's only in the local version. The first live forecast can't be scored until November 8.

## Key source files

1. `midterm/analyze_topic.py`: the main pipeline, from measuring to writing and checking
2. `midterm/topic_trends.py`: collects autocomplete phrases and news
3. `midterm/eval/score_predictions.py`: makes forecasts and scores them later
4. `midterm/eval/jev_agreement.py`: the blind test of Jev
5. `midterm/dashboard_template.html`: the web page
