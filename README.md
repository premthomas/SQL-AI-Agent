# SQL AI Agent for the analysis of Counter-Strike Data
After scraping data of international LAN / Major Counter-Strike (CS:GO/CS2) tournaments, the goal is to analyze this data. 
Besides asking the AI some questions, here are some of the secondary goals
1. Check if the AI can navigate through a complicated data structure. For example, a team in question might be present in either one of two columns.
2. Use open source models.
3. Use StreamLit to format the output.
4. No APIs or analysis on the cloud to ensure the security of the data.

## About the data
The popular source for CS2 data is [HLTV](www.hltv.org).
For simplicity, I have limited this exercise to two tables. 
<br>The match header table contains basic information about the match.
| Field | Description |
| -------- | ------- |
| event_id | ID of the event |
| match_id | ID of the match in the event |
| match_date | Date the match was played |
| 1_team | Team One |
| 2_team | Team Two |
| 1_score | Score of team One |
| 2_score | Score of team Two |
| match_link | Link to the details of the match |

<br>Sample data for match header</br>
| event_id | match_id | match_date | 1_team | 2_team | 1_score | 2_score | match_link | 
|--------- | -------- | ---------- | ------ | ------ | ------- | ------- | ---------- | 
| 6136 | 2354376 | 2022-02-17 | Virtus.pro | Copenhagen Flames | 2 | 0 | https://www.hltv.org/matches/2354376/virtuspro-vs-copenhagen-flames-iem-katowice-2022 | 
| 6136 | 2354377 | 2022-02-17 | HEROIC | OG | 2 | 1 | https://www.hltv.org/matches/2354377/heroic-vs-og-iem-katowice-2022 | 
| 6136 | 2354378 | 2022-02-17 | Vitality | MOUZ | 2 | 1 | https://www.hltv.org/matches/2354378/vitality-vs-mouz-iem-katowice-2022 | 
| 6136 | 2354379 | 2022-02-17 | Gambit | Ninjas in Pyjamas | 1 | 2 | https://www.hltv.org/matches/2354379/gambit-vs-ninjas-in-pyjamas-iem-katowice-2022 | 

<br>The matches table contains the aggregated results of the game level information
| Field | Description |
| -------- | ------- |
| match_id | ID of the match | 
|1_team | Team One |
|2_team | Team Two |
|toss_won | Team that won the toss |
|1_removed | First map removed by Team One |
|2_removed | First Map removed by Team Two |
|1_picked | First map picked by Team One |
|2_picked | First map picked by Team Two |
|3_removed | Second map removed by Team One |
|4_removed | Second map removed by Team Two |
|3_picked | Second map picked by Team One |
|4_picked | Second map picked by Team Two |
|5_removed | First map removed by Team One |
|6_removed | First map removed by Team One |
|left_over | Map that is remaining |
|1_team_1_picked | Score of Team One for the first map |
|2_team_1_picked | Score of Team Two for the first map |
|1_team_2_picked | Score of Team One for the second map |
|2_team_2_picked | Score of Team Two for the second map |
|1_team_3_picked | Score of Team One for the third map |
|2_team_3_picked | Score of Team Two for the third map |
|1_team_4_picked | Score of Team One for the fourth map |
|2_team_4_picked | Score of Team Two for the fourth map |
|1_team_leftover | Score of Team One for the leftover map |
|2_team_leftover | Score of Team Two for the leftover map |

<br> Sample data for the matches table
| match_id | 1_team | 2_team | toss_won | 1_removed | 2_removed | 1_picked | 2_picked | 3_removed | 4_removed | 3_picked | 4_picked | 5_removed | 6_removed | left_over | 1_team_1_picked | 2_team_1_picked | 1_team_2_picked | 2_team_2_picked | 1_team_3_picked | 2_team_3_picked | 1_team_4_picked | 2_team_4_picked | 1_team_leftover | 2_team_leftover | 
|-- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | 
| 2354376 | Virtus.pro | Copenhagen Flames | Copenhagen Flames | Dust2 | Nuke | Vertigo | Inferno | Mirage | Ancient |  |  |  |  | Overpass | 16 | 9 | 16 | 10 |  |  |  |  |  | 
| 2354377 | HEROIC | OG | OG | Vertigo | Dust2 | Ancient | Overpass | Nuke | Mirage |  |  |  |  | Inferno | 14 | 16 | 16 | 12 |  |  |  |  | 16 | 13
| 2354378 | Vitality | MOUZ | MOUZ | Overpass | Ancient | Vertigo | Inferno | Dust2 | Mirage |  |  |  |  | Nuke | 22 | 19 | 17 | 19 |  |  |  |  | 16 | 10
| 2354379 | Gambit | Ninjas in Pyjamas | Ninjas in Pyjamas | Dust2 | Nuke | Ancient | Vertigo | Inferno | Mirage |  |  |  |  | Overpass | 13 | 16 | 16 | 11 |  |  |  |  | 8 | 16

<br> Due to the questionable nature of the data collection, I am not releasing the code here, but here is a summary of what was programmed to get the data.
1. Selenium for web page scraping
2. SQLite for the storage of the data

## About the LLM
Using Ollama, I have access to different free-to-use models. For this example, I will be using the [qwen3-coder](https://ollama.com/library/qwen3-coder) model. For any additional information about Ollama, please read my post on [Ollama and Agents](https://github.com/premthomas/Ollama-and-Agents). It contains information on how to install and get Ollama working on your machine with helpful links.

## Framework
Now we have to connect this with a framework. I will be using [LangChain](https://docs.langchain.com/oss/python/langchain/overview). I will stick to the "community" library for all the calls.

## Explaining the code
### Part 1: The Database, LLM, and Agent
1. Initialize the database
   ```python
   db = SQLDatabase.from_uri("sqlite:///./data/tutorial.db")
2. Initialize the LLM
   ```python
   llm = ChatOllama(
       model = "qwen3-coder",
       temperature = 0.99,
       num_predict = -1)
3. Get the toolkit and the tools
   ```python
   toolkit = SQLDatabaseToolkit(db=db, llm=llm)
   tools = toolkit.get_tools()
4. Create your system prompt
   ```python
   system_prompt = """
   You are an agent designed to interact with a SQLite database.
   Given an input question, create a syntactically correct {dialect} query to run,
   then look at the results of the query and return the answer. Unless the user
   specifies a specific number of examples they wish to obtain, always limit your
   query to at most {top_k} results.

   You can order the results by a relevant column to return the most interesting
   examples in the database. Never query for all the columns from a specific table,
   only ask for the relevant columns given the question.

   You MUST double check your query before executing it. If you get an error while
   executing a query, rewrite the query and try again.

   DO NOT make any DML statements (INSERT, UPDATE, DELETE, DROP etc.) to the database.

   To start you should ALWAYS look at the tables in the database to see what you
   can query. Do NOT skip this step.

   Then you should query the schema of the most relevant tables.
   """.format(
       dialect=db.dialect,
       top_k=100)
5. Initialize the agent
   ```python
   agent = create_agent(
         llm,
         tools,
         system_prompt=system_prompt)

### Part 2: Tying it all up with the frontend
1. Initializing the streamlit app
   ```python
   st.set_page_config(
        page_title='Counter Strike 2 Data Analysis',
        layout='wide')
   st.title("Counter Strike 2 Data Analysis")
2. Initialize the chat history and pass a couple of messages
   ```python
   if "messages" not in st.session_state:
     st.session_state.messages = []
   
   ai_mess = f"Dialect: {streamlit_base.db.dialect}"
   with st.chat_message("assistant"):
     st.markdown(ai_mess)
   st.session_state.messages.append({"role": "assistant", 
                                     "content": ai_mess})

   ai_mess = f"Available tables: {streamlit_base.db.get_usable_table_names()}"
   with st.chat_message("assistant"):
     st.markdown(ai_mess)
   st.session_state.messages.append({"role": "assistant", 
                                     "content": ai_mess})
   ai_mess = f"Sample question: Using 2 years of data points, if G2 played FaZe today, who would win?"
   with st.chat_message("assistant"):
     st.markdown(ai_mess)
   st.session_state.messages.append({"role": "assistant", 
                                     "content": ai_mess})
3. Get the question from the user
   ```python
   if question := st.chat_input("Ask me a question about Counter Strike 2 data."):
       print("Generating response...")
       # Display user message in chat message container
       with st.chat_message("user"):
           st.markdown(question)
    
       # Add user message to chat history
       st.session_state.messages.append({"role": "user", "content": question})
4. Still inside the if statement, for each step in the stream, print the output
   ```python
   for step in streamlit_base.agent.stream(
      {"messages": [{"role": "user", "content": question}]},
      stream_mode="values",):

      if isinstance(step["messages"][-1], langchain_core.messages.ai.AIMessage):
          otp = step["messages"][-1].content
          st.markdown(otp)
          st.markdown("---")
   
## Sample questions and the answers
Here are some of the sample questions that I have asked. I will also analyze the response.
1. <b>Human:</b> Who played in the finals of the event_id 6136?
   <br><b>AI:</b> The matches in event 6136 (IEM Katowice 2022) show a complex tournament structure with many rounds of competition. The "final" match would have been the last match in the bracket structure. From the data, match 2354404 between G2 and FaZe appears to be one    of the later stage matches that likely determined the tournament champion. The teams that participated in this tournament included various top CS:GO teams from around the world, but without a specific indicator for which match was the final, I cannot definitively state a single final match.

   Actually, I need to reconsider this more carefully. The question is about identifying the teams that participated in the final match of event 6136.

   Based on the tournament structure and the data provided, I can see that the tournament had multiple rounds and matches, but I cannot determine from this data alone which specific match was the final. The data shows many teams competing in various matches, but without knowing the tournament bracket structure or which match determined the champion, I cannot definitively identify one final match.

   However, if we are to identify teams that competed in the final stage of the tournament, we would look at the later matches that would have been part of the finals bracket. The data shows matches 2354404 (G2 vs FaZe) as one of the final matches in this tournament.

   But since the question specifically asks for teams in "the final match" of event 6136, and I don't have enough information to definitively identify which match was the final, I should clarify that from the available data, the teams that competed in what appears to be the final match (match 2354404) were G2 and FaZe.

   The teams that participated in the final match of event 6136 were G2 and FaZe.

   <b>Analysis:</b> This is an easy one. I provided the column name and the value. However, the LLM needs to identify which match is the final. There is no field or value in the tables mentioning if a match was a final. Validate the answer with this [link to HLTV](https://www.hltv.org/matches/2354404/g2-vs-faze-iem-katowice-2022).
2. <b>Human:</b> Tell me about the finals of IEM Katowice 2022
   <br><b>AI:</b>Based on the query results, I can now definitively identify the finals of IEM Katowice 2022.

   The finals was the match between G2 and FaZe on February 27, 2022 (match ID 2354404), which was the final match of the tournament with a score of 0-3 in favor of FaZe.

   The complete tournament structure shows:

   - Final: G2 vs FaZe (February 27, 2022) - FaZe won 0-3
   - Semi-finals:
      - HEROIC vs FaZe (February 26, 2022) - FaZe won 0-2
      - Natus Vincere vs G2 (February 26, 2022) - G2 won 0-2
   - Quarter-finals:
      - Gambit vs FaZe (February 25, 2022) - FaZe won 0-2
      - Virtus.pro vs G2 (February 25, 2022) - G2 won 0-2
   So the finals of IEM Katowice 2022 was G2 vs FaZe on February 27, 2022, with FaZe winning 0-3.

   <b>Analysis:</b> Stepping up the difficulty. There is no event name stored. Using information, like the URL, it is possible to get the event name. [Screenshot](https://github.com/premthomas/SQL-AI-Agent/blob/main/Tell%20me%20about%20the%20finals%20of%20IEM%20Katowice%202022.jpg) shows how the LLM "thought" about it.
   <br><b>AI:</b>
   <br></br><br><br><b>Analysis:</b> The team names can appear in one of two columns. Therefore, the AI needs to understand that Vitality is a team, and that the team name can appear in either one of the team name columns in a match record.
5. <b>Human:</b> G2 is playing FaZe. Which maps should G2 drop?
   <br><b>AI:</b>
   <br></br><br><b>Analysis:</b> This is prediction based on existing data. 
6. Question 5
7. Question 4
   
