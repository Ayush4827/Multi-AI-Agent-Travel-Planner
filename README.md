# AI Travel Planner (CrewAI Agents)

This is a small project I built while learning about AI agents using CrewAI. The idea: instead of one AI trying to plan a whole trip by itself, split the job into a few "agents" that each handle one part, and have them pass their work along to the next one — kind of like a small team where everyone has one job.

I'm still new to this (started learning agents about a week ago), so this is more of a learning project than a polished product. Sharing the code and what I learned building it.

## What it does

You give it a trip: where you're going, the dates, how many people, and budgets for transport/stay/activities. It gives you back a full day-by-day travel plan.

## The agents

I used 5 agents, each with its own job:

1. **Transportation Researcher** — looks up how to get from A to B (flight/train/bus etc.) within the budget, for both the onward and return trip.
2. **Accommodation Researcher** — looks up places to stay based on budget, number of people, and how many nights.
3. **Activities Researcher** — finds things to do and food to try at the destination.
4. **Itinerary Researcher** — takes what the first three agents found and turns it into an actual day-by-day plan.
5. **QA Researcher** — goes over everything the other agents produced and checks if it actually makes sense (budget not exceeded, no overlapping activities, timings realistic, etc.) before giving a final "approved" or "needs fixing."

They run one after another (not all at once), and CrewAI lets me pass the output of the first three agents into the itinerary agent using its `context` parameter, so it doesn't have to start from scratch.

For the actual LLM, I used Google's Gemini 2.5 Flash. For the research agents, I gave them a web search tool (Tavily) so they can look up real, current info instead of just guessing from training data.

## Built with

- Python (Jupyter Notebook)
- [CrewAI](https://github.com/crewAIInc/crewAI) for the agent/task setup
- Google Gemini 2.5 Flash as the LLM
- Tavily API for web search

## Running it yourself

1. Install the packages:
   ```bash
   pip install crewai crewai-tools tavily-python python-dotenv
   ```

2. You'll need your own API keys — one from Google AI Studio (Gemini) and one from Tavily. **Don't paste them directly into the notebook.** Put them in a `.env` file instead:
   ```
   GOOGLE_API_KEY=your_key_here
   TAVILY_API_KEY=your_key_here
   ```

3. Load them in the notebook like this instead of hardcoding:
   ```python
   import os
   from dotenv import load_dotenv
   load_dotenv()

   llm = LLM(model="gemini-2.5-flash", google_api_key=os.getenv("GOOGLE_API_KEY"), temperature=0.3)
   ```

4. Make sure `.env` is in your `.gitignore` so you don't accidentally commit your keys.

5. Run the notebook and fill in your trip details at the bottom:
   ```python
   result = await crew.kickoff_async(
       inputs={
           "from_place": "Delhi",
           "to_place": "Manali",
           "going_date": "2026-10-10",
           "returning_date": "2026-10-15",
           "number_of_adults": 2,
           "number_of_kids": 1,
           "transportation_budget": 15000,
           "accommodation_budget": 20000,
           "activities_budget": 10000
       }
   )
   print(result.raw)
   ```

## What I'd improve next

- Add basic input validation (e.g. make sure the return date is after the going date).
- Clean up some typos in the agent descriptions.
- Maybe save the output as a PDF instead of just printing it.
- Try a simple front end (Streamlit?) so I don't have to run cells manually every time.

## Notes to self

This was my first time actually building something with multi-agent orchestration instead of just calling an LLM once. Biggest thing I learned: how to pass context between tasks so later agents can use earlier agents' output, instead of every agent working in isolation.
