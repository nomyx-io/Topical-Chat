# Topical-Chat
We introduce Topical-Chat, a knowledge-grounded
human-human conversation dataset where the underlying
knowledge spans 8 broad topics and conversation
partners don’t have explicitly defined roles.

Topical-Chat broadly consists of two types of files:

- Conversations: JSON files containing conversations between pairs of
Amazon Mechanical Turk workers.
- Reading Sets: JSON files containing knowledge sections rendered as
reading content to the Turkers having conversations.

We provide a simple script, `build.py`, to build the
reading sets for the dataset, by making API calls
to the relevant sources of the data.

For detailed information about the dataset, modeling
benchmarking experiments and evaluation results,
please refer to our [paper](https://m.media-amazon.com/images/G/01/amazon.jobs/3079_Paper._CB1565131710_.pdf).

## Prerequisites

After cloning this repo, please run the following commands,
preferably after creating a virtual environment:

```
cd src/
pip install -r requirements.txt
```

Please create your own Reddit API
[credentials](https://www.reddit.com/wiki/api),
and manually add them to `src/reddit/prawler.py`

The scripts in this repo have been tested with Python 3.7
and we recommend using Python 3.7.

## Build

Run `python build.py`, after having manually added your
own Reddit credentials in `src/reddit/prawler.py` and creating a `reading_sets/post-build/` directory.

`build.py` will read each file in `reading_sets/pre-build/`,
create a replica JSON with the exact same name and the actual reading
sets included in `reading_sets/post-build/`.

## Dataset

### Statistics:
|       Stat            | Train | Valid Freq. | Valid Rare | Test Freq. | Test Rare | All |
| ----                  | ----  |    ----     |    ----    |   ----     |   ----    |  ----   |
|# of conversations    | 8628  |    539      |    539     |   539      |   539     |  10784  |
|# of utterances       | 188378 |   11681    |    11692   |   11760    |   11770   |  235281 |
|average # of turns per conversation  | 21.8 |    21.6    |   21.7     |   21.8    |   21.8  |  21.8   |
|average length of utterance    | 19.5  |    19.8      |    19.8     |   19.5      |   19.5     |  19.6   |

### Split:
The data is split into 5 distinct groups: *train*, *valid frequent*,
*valid rare*, *test frequent* and *test rare*. The frequent set
contains entities frequently seen in the training set. The rare set
contains entities that were infrequently seen in the training set.

### Configuration Type:
For each conversation to be collected, we applied a random
knowledge configuration from a pre-defined list of configurations,
to construct a pair of reading sets to be rendered to the partnered
Turkers. Configurations were defined to impose varying degrees of
knowledge symmetry or asymmetry between partner Turkers, leading to
the collection of a wide variety of conversations.

![Reading sets for Turkers 1 and 2 in Config A](images/configA.png)

![Reading sets for Turkers 1 and 2 in Config B](images/configB.png)

![Reading sets for Turkers 1 and 2 in Config C&D](images/configCD.png)


### Conversations:

**Each JSON file in conversations/ has the following
format:**
```
{
<conversation_id>: {
	“article_url”: <article url>,
	“config”: <config>, # one of A, B, C, D
	“content”: [ # ordered list of conversation turns
		{ 
		“agent”: “agent_1”, # or “agent_2”,
		“message” : <message text>,
		“sentiment”: <text>,
		“knowledge_source” : [“AS1”, “Personal Knowledge”, ...],
		“turn_rating”: “Poor”,
		},…
	],
	“conversation_rating”: {
		“agent_1”: “Good”,
		“agent_2”: “Excellent”
		}
},…
}
```
- conversation_id: A unique identifier for a conversation in Topical-Chat
- article_url: URL pointing to the Washington Post article associated
with a conversation
- config: The knowledge configuration applied to obtain a pair of
reading sets for a conversation
- content: An ordered list of conversation turns
	- agent: An identifier for the Turker who generated the message
	- message: The message generated by the agent
	- sentiment: Self-annotation of the sentiment of the message
	- knowledge_source: Self-annotation of the section within
	the agent's reading set used to generate this message
	- turn_rating: Partner-annotation of the quality of the message
- conversation_rating: Self-annotation of the quality of the conversation
	- agent_1: Rating of the conversation by Turker 1
	- agent_2: Rating of the conversation by Turker 2

### Reading Sets:

**Each JSON file in reading_sets/post-build/ has the
following format:**
```
{
<conversation_id> : {
	“config” : <config>,
    “agent_1”: {
	    “FS1”: {
		 “entity”: <entity name>,
		 “shortened_wiki_lead_section”: <section text>,
		 “fun_facts”: [ <fact1_text>, <fact2_text>,…]
		    },
	    “FS2”:…
                    },
        ....
        },
    “agent_2”: {
	    “FS1”: {
		 “entity”: <entity name>,
		 “shortened_wiki_lead_section”: <section text>,
		 “fun_facts”: [ <fact1_text>, <fact2_text>,…],
	            },
	    “FS2”:…
                    },
        ...
        },
    “article”: {
		“url”: <url>,
		“headline” : <headline text>,
		“AS1”: <section 1 text>,
		“AS2”: <section 2 text>,
		“AS3”: <section 3 text>,
		“AS4”: <section 4 text>
	    }
	},
…
}
```
- conversation_id: A unique identifier for a conversation in
Topical-Chat
- config: The knowledge configuration applied to obtain a pair of
reading sets for a conversation
- agent_{1/2}: Contains the factual sections in this agent's reading set
	- FS{1/2/3}: Identifier for a factual section
		- entity: A real-world entity
		- shortened_wiki_lead_section: A shortened version of the
		Wikipedia lead section of the entity
		- summarized_wiki_lead_section: A (TextRank) summarized version
		of the Wikipedia lead section of the entity
		- fun_facts: Crowdsourced and manually curated fun facts about
		the entity from Reddit's r/todayilearned subreddit
- article: A Washington Post article common to both partners'
reading sets
    - url: URL pointing to the Washington Post article associated
with a conversation
    - headline: The headline of the Washington Post article
	- AS{1/2/3/4}: A chunk of the body of the Washington Post article

### Wikipedia Data:
**src/wiki.json has the following format:**

```
{
  "shortened_wiki_lead_section": {
    <shortened wiki lead section text>: <unique_identifier>,
    <shortened wiki lead section text>: <unique_identifier>
  },
  "summarized_wiki_lead_section": {
    <summarized wiki lead section text>": <unique_identifier>,
    <summarized wiki lead section text>": <unique_identifier>
  }
}
```

`build.py` puts data from `wiki.json` into the relevant reading
sets.

## Citation
If you use Topical-Chat in your work, please cite with the following:
```
@inproceedings{gopalakrishnan2019topical,
  author={Karthik Gopalakrishnan and Behnam Hedayatnia and Qinlang Chen and Anna Gottardi and Sanjeev Kwatra and Anu Venkatesh and Raefer Gabriel and Dilek Hakkani-Tür},
  title={{Topical-Chat: Towards Knowledge-Grounded Open-Domain Conversations}},
  year=2019,
  booktitle={Proc. Interspeech 2019},
  pages={1891--1895},
  doi={10.21437/Interspeech.2019-3079},
  url={http://dx.doi.org/10.21437/Interspeech.2019-3079}
}
```

```
Gopalakrishnan, Karthik, et al. "Topical-Chat: Towards Knowledge-Grounded Open-Domain Conversations.", Proc. INTERSPEECH 2019
```

## Acknowledgements
We thank Anju Khatri, Anjali Chadha and
Mohammad Shami for their help with the public release of
the dataset. We thank Jeff Nunn and Yi Pan for their
early contributions to the dataset collection.
