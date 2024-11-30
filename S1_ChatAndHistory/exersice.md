# Exercise: Configuring Language Models and Prompts

## Objective
To practice configuring language models (Cohere, Groq) and creating prompt templates to generate a story.

## Instructions
Follow the steps below:

1. **Create `actor_picker` Function**:
   - Write a function named `actor_picker()` that randomly returns an actor's name. You can use Python's random generator to select an actor.

2. **Create `location_picker` Function**:
   - Write a function named `location_picker()` that randomly returns the name of a location.

3. **Create `theme_picker` Function**:
   - Write a function named `theme_picker()` that randomly returns a theme.

4. **Configure and Use Cohere Model**:
   - Set up the Cohere model using `ChatCohere`.
   - Create a prompt template that generates a story centered around the chosen **Actor**, **Location**, and **Theme** using the Cohere model.
   - Print the detailed story output from the model.

5. **Configure and Use Groq Model**:
   - Set up the Groq model using `ChatGroq`.
   - Create a prompt template to generate a story using the Groq model.
   - Print the detailed response from the model.
     
6. **Add InMemoryChatMessageHistory to store conversation**:
   - Use 'InMemoryChatMessageHistory', to store history and allow user customize the story.
   - You can pass the story created with prompt saying 'change the given story according to user input'.

6. **Execution**:
   - When you run the code, it should randomly pick an actor, location, and theme, then generate a story around these elements. Then it should ask for user input to customize the story. You may include heroines in the story for added detail.

## Example Output

### Selected Elements
- **Actor**: Chris Evans
- **Location**: New York
- **Theme**: Adventure

---

### Story from Cohere Model
Chris Evans found himself in the bustling streets of New York, surrounded by towering skyscrapers and the endless hum of city life. One evening, while exploring an old library, he stumbled upon a mysterious map hidden in a dusty, leather-bound book. The map hinted at an ancient artifact buried deep beneath the city, a relic that could change the course of history. Determined to uncover the truth, Chris embarked on a thrilling adventure, navigating secret tunnels, facing unexpected dangers, and relying on his courage to guide him through the labyrinthine passages beneath the city. With each step, he realized this journey would not only test his strength but also reveal secrets of New York’s hidden past. Would he find the artifact? Only time would tell.

---

### Story from Groq Model
In the heart of New York, where lights shimmer like stars, Chris Evans was drawn into an unexpected journey. The city had always held secrets, but tonight it felt alive, whispering to him through the cold night air. Following an old legend, he ventured into abandoned subway stations and hidden alleyways, guided only by a cryptic poem and his instinct. Each clue led him closer to a fabled artifact said to hold immense power. Alongside a mysterious heroine he met along the way, Chris faced twists and turns that tested their resolve. In the end, the adventure would unveil not just the artifact but a part of himself he never knew existed.

---


import os
import configparser
from langchain_core.output_parsers import StrOutputParser
from langchain_groq import ChatGroq
from langchain_cohere import ChatCohere
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.messages import HumanMessage, SystemMessage

config = configparser.ConfigParser()
config.read('config.ini')
groq = config['groq']
cohere = config['cohere']
os.environ['GROQ_API_KEY'] = groq.get('GROQ_API_KEY')
os.environ['COHERE_API_KEY'] = cohere.get('COHERE_API_KEY')

def actor_picker():
    messages = [
    SystemMessage(content='You have to pickup any random Bollywood actor from 1 to 5 and only give a actor name'),
    HumanMessage(content=input("Enter a number from 1 to 5 choose a random Bollywood actor :- "))
    ]
    parser = StrOutputParser()
    model = ChatGroq(model="llama3-8b-8192")
    chain=model | parser
    response= chain.invoke(messages)
    print(response)
    return response
def location_picker():
    messages = [
    SystemMessage(content='You have to pick any random location and give different location and only give a location name'),
    #HumanMessage(content=input("Enter a number from 1 to 5 choose a random location:- "))
    ]
    parser = StrOutputParser()
    model = ChatGroq(model="llama3-8b-8192")
    chain=model | parser
    response= chain.invoke(messages)
    print(response)
    return response
def theme_picker():
    messages = [
    SystemMessage(content='You have to pickup any random theme for movie and only give a theme'),
    #HumanMessage(content=input("Enter a number from 1 to 5 choose a random theme:- "))
    ]
    parser = StrOutputParser()
    model = ChatGroq(model="llama3-8b-8192")
    chain=model | parser
    response= chain.invoke(messages)
    print(response)
    return response
actor=actor_picker()
location=location_picker()
theme=theme_picker()
parser = StrOutputParser()

system_template = "Return an output in string form as a story reading the output for the input three functions. Please provide a {{size}} related response."

prompt_template = ChatPromptTemplate.from_messages(
    [
        ("system", system_template),
        ("user", 'Create a story for a movie in which the actor is {actor}, the location for movie is {location} and the theme for the movie is {theme}.Tell the genre, location and starring for the movie and then display story.')  # Use string representation of my_list here
    ]
)
def scriptwriter():
    model_groq = ChatGroq(model="llama3-8b-8192")
    chain = prompt_template | model_groq | parser

    final_story_output = chain.invoke({"actor":actor,"location":location,"theme":theme})

    print(final_story_output)
    return final_story_output
script=scriptwriter()
# story=story_generator()
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser


model = ChatGroq(model="llama3-8b-8192")

from langchain_core.chat_history import(InMemoryChatMessageHistory)
import asyncio
store=InMemoryChatMessageHistory()

async def History_Store():
    store.add_message(HumanMessage(content=f'{script}'))
    messages = await store.aget_messages()
    response = model.invoke(messages)
    print(response.content)
    store.add_message(SystemMessage(content=response.content))
    
async def New_Story():
    await asyncio.sleep(2)  
    system_message_content = "Please change the movie story according to the changes in the given input"
    store.add_message(SystemMessage(content=system_message_content))
    user_input = input("Tell if you want to make changes to in your story:- ")
    store.add_message(HumanMessage(content=user_input))
    messages = await store.aget_messages()
    response = model.invoke(messages)
    print("Model Response:", response.content)

    store.add_message(SystemMessage(content=response.content))

async def final_Story():
    await History_Store()
    await New_Story()
asyncio.run(final_Story())
