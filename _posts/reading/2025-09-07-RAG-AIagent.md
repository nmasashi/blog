---
layout: post
title: "RAG・AIエージェント[実践]入門"
date: 2025-08-05
categories: reading python RAG LangChain LangGraph
---

![image]({{site.baseurl}}/images/reading/rag_aiagent_nyuumon/bookcover.jpg)

この本読んだのでメモ

## 第 2 章 OpneAI チャットの API の基礎

### 2.3 入出力の長さの制限や料金に影響する「トークン」

#### トークン

- GPT-4o などのモデルはテキストを「トークン」という単位に分割する
- トークンは単語と一致するわけはない。「ChatGPT」なら「Chat」と「GPT」になったりする。
- 日本語での表現は英語よりトークン数が必要な傾向がある。
- しかし、最近は改善されており 1.X 倍程度にまでなってるっぽい

### 2.5 Chat Completions API のハンズオン

しょっぱなハンズオンでいきなりエラー。。。

[先人の知恵](https://qiita.com/hatman621221/items/fafaf4ac0e906616e8bd)により解消

実施したコード

```python
from openai import OpenAI

client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "こんにちは！私はジョンと言います！"},
    ],
)
print(response.to_json(indent=2))

```

出力

```json
{
	"id": "chatcmpl-CDAZA0FC37Ey7HswzEK1W5LfFGimx",
	"choices": [
		{
			"finish_reason": "stop",
			"index": 0,
			"logprobs": null,
			"message": {
				"content": "こんにちは、ジョンさん！お話しできてうれしいです。今日はどんなことを話しましょうか？",
				"refusal": null,
				"role": "assistant",
				"annotations": []
			}
		}
	],
	"created": 1757254916,
	"model": "gpt-4o-mini-2024-07-18",
	"object": "chat.completion",
	"service_tier": "default",
	"system_fingerprint": "fp_8bda4d3a2c",
	"usage": {
		"completion_tokens": 26,
		"prompt_tokens": 25,
		"total_tokens": 51,
		"prompt_tokens_details": {
			"cached_tokens": 0,
			"audio_tokens": 0
		},
		"completion_tokens_details": {
			"reasoning_tokens": 0,
			"audio_tokens": 0,
			"accepted_prediction_tokens": 0,
			"rejected_prediction_tokens": 0
		}
	}
}
```

会話を続けてみる。
先ほどの返答を追加し、こちらから質問してみる。

```python
from openai import OpenAI

client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "こんにちは！私はジョンと言います！"},
        {"role": "assistant", "content": "こんにちは、ジョンさん！お話しできてうれしいです。今日はどんなことを話しましょうか？"},
        {"role": "user", "content": "私の名前がわかりますか？"},
    ],
)
print(response.to_json(indent=2))
```

返答

```json
{
	"id": "chatcmpl-CDAwgGCqIRuTKeNSmkRFgvXpRx69x",
	"choices": [
		{
			"finish_reason": "stop",
			"index": 0,
			"logprobs": null,
			"message": {
				"content": "はい、あなたの名前はジョンさんですね。どうぞ他にお話ししたいことがあれば教えてください！",
				"refusal": null,
				"role": "assistant",
				"annotations": []
			}
		}
	],
	"created": 1757256374,
	"model": "gpt-4o-mini-2024-07-18",
	"object": "chat.completion",
	"service_tier": "default",
	"system_fingerprint": "fp_8bda4d3a2c",
	"usage": {
		"completion_tokens": 28,
		"prompt_tokens": 68,
		"total_tokens": 96,
		"prompt_tokens_details": {
			"cached_tokens": 0,
			"audio_tokens": 0
		},
		"completion_tokens_details": {
			"reasoning_tokens": 0,
			"audio_tokens": 0,
			"accepted_prediction_tokens": 0,
			"rejected_prediction_tokens": 0
		}
	}
}
```

ちゃんとした返答がえられる。また、`stream=True`というパラメータを付けるとストリーミング形式で少しずつ応答を得ることができる（chatGPT でちょっとづつ回答がでてくるの同じイメージ）。

このようにするとレスポンスを JSON に指定できる

```python
from openai import OpenAI

client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {
            "role": "system",
            "content": '人物一覧を次のJSON形式で出力してください。\n{"people": ["aaa", "bbb"]}',
        },
        {
            "role": "user",
            "content": "昔々あるところにおじいさんとおばあさんがいました",
        },
    ],
    response_format={"type": "json_object"},
)
print(response.choices[0].message.content)
```

### 2.6 Functuion calling

Functuion calling: 利用可能な関数を LLM に伝えておいて、LLM に使いたい関数を判断させる。実行は LLM がするのではなく API サーバー等で行う。

例：
天気を取得する用の関数を用意

```python
import json

def get_current_weather(location, unit="fahrenheit"):
    if "tokyo" in location.lower():
        return json.dumps({"location": "Tokyo", "temperature": "10", "unit": unit})
    elif "san francisco" in location.lower():
        return json.dumps(
            {"location": "San Francisco", "temperature": "72", "unit": unit}
        )
    elif "paris" in location.lower():
        return json.dumps({"location": "Paris", "temperature": "22", "unit": unit})
    else:
        return json.dumps({"location": location, "temperature": "unknown"})
```

どのようなツール（関数）があるのかを定義。説明やパラメータを定義している。

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_current_weather",
            "description": "Get the current weather in a given location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "The city and state, e.g. San Francisco, CA",
                    },
                    "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
                },
                "required": ["location"],
            },
        },
    }
]
```

使うことができる関数の一覧を取得する

```python
from openai import OpenAI

client = OpenAI()

messages = [
    {"role": "user", "content": "東京の天気はどうですか？"},
]

response = client.chat.completions.create(
    model="gpt-4o",
    messages=messages,
    tools=tools,
)
print(response.to_json(indent=2))
```

出力結果

```json
{
	"id": "chatcmpl-CDtTtHvdUdvYAQW5NKZlcI3mRhmJd",
	"choices": [
		{
			"finish_reason": "tool_calls",
			"index": 0,
			"logprobs": null,
			"message": {
				"content": null,
				"refusal": null,
				"role": "assistant",
				"annotations": [],
				"tool_calls": [
					{
						"id": "call_xRszPzLPcCIEEUvjeWItbij2",
						"function": {
							"arguments": "{\"location\":\"東京\"}",
							"name": "get_current_weather"
						},
						"type": "function"
					}
				]
			}
		}
	],
	"created": 1757427569,
	"model": "gpt-4o-2024-08-06",
	"object": "chat.completion",
	"service_tier": "default",
	"system_fingerprint": "fp_1827dd0c55",
	"usage": {
		"completion_tokens": 15,
		"prompt_tokens": 81,
		"total_tokens": 96,
		"completion_tokens_details": {
			"accepted_prediction_tokens": 0,
			"audio_tokens": 0,
			"reasoning_tokens": 0,
			"rejected_prediction_tokens": 0
		},
		"prompt_tokens_details": {
			"audio_tokens": 0,
			"cached_tokens": 0
		}
	}
}
```

使用する必要がある tool 名`get_current_weather`とその引数`{\"location\":\"東京\"}`がわかる。

使用する必要がある tool の情報を保存

```python
response_message = response.choices[0].message
messages.append(response_message.to_dict())
```

tool の実行はこちらで行い、結果を`messages`に格納する

```python
available_functions = {
    "get_current_weather": get_current_weather,
}

# 使いたい関数は複数あるかもしれないのでループ
for tool_call in response_message.tool_calls:
    # 関数を実行
    function_name = tool_call.function.name
    function_to_call = available_functions[function_name]
    function_args = json.loads(tool_call.function.arguments)
    function_response = function_to_call(
        location=function_args.get("location"),
        unit=function_args.get("unit"),
    )
    print(function_response)

    # 関数の実行結果を会話履歴としてmessagesに追加
    messages.append(
        {
            "tool_call_id": tool_call.id,
            "role": "tool",
            "name": function_name,
            "content": function_response,
        }
    )
```

最終的なメッセージの内容はこんな感じ(出力方法：`print(json.dumps(messages, ensure_ascii=False, indent=2))`)

```json
[
	{
		"role": "user",
		"content": "東京の天気はどうですか？"
	},
	{
		"content": null,
		"refusal": null,
		"role": "assistant",
		"annotations": [],
		"tool_calls": [
			{
				"id": "call_xRszPzLPcCIEEUvjeWItbij2",
				"function": {
					"arguments": "{\"location\":\"東京\"}",
					"name": "get_current_weather"
				},
				"type": "function"
			}
		]
	},
	{
		"tool_call_id": "call_xRszPzLPcCIEEUvjeWItbij2",
		"role": "tool",
		"name": "get_current_weather",
		"content": "{\"location\": \"\\u6771\\u4eac\", \"temperature\": \"unknown\"}"
	},
	{
		"content": null,
		"refusal": null,
		"role": "assistant",
		"annotations": [],
		"tool_calls": [
			{
				"id": "call_xRszPzLPcCIEEUvjeWItbij2",
				"function": {
					"arguments": "{\"location\":\"東京\"}",
					"name": "get_current_weather"
				},
				"type": "function"
			}
		]
	},
	{
		"tool_call_id": "call_xRszPzLPcCIEEUvjeWItbij2",
		"role": "tool",
		"name": "get_current_weather",
		"content": "{\"location\": \"\\u6771\\u4eac\", \"temperature\": \"unknown\"}"
	}
]
```

このメッセージを使用して Chat Completions API にリクエストをしてみる。

```python
second_response = client.chat.completions.create(
    model="gpt-4o",
    messages=messages,
)
print(second_response.to_json(indent=2))
```

結果

```json
{
	"id": "chatcmpl-CDtcZgpzqbLOFGkzIHMfZcRqznhUZ",
	"choices": [
		{
			"finish_reason": "stop",
			"index": 0,
			"logprobs": null,
			"message": {
				"content": "申し訳ありませんが、現在の東京の天気情報を取得することができませんでした。他の方法で確認していただくか、別の質問があればお知らせください。",
				"refusal": null,
				"role": "assistant",
				"annotations": []
			}
		}
	],
	"created": 1757428107,
	"model": "gpt-4o-2024-08-06",
	"object": "chat.completion",
	"service_tier": "default",
	"system_fingerprint": "fp_cbf1785567",
	"usage": {
		"completion_tokens": 40,
		"prompt_tokens": 100,
		"total_tokens": 140,
		"completion_tokens_details": {
			"accepted_prediction_tokens": 0,
			"audio_tokens": 0,
			"reasoning_tokens": 0,
			"rejected_prediction_tokens": 0
		},
		"prompt_tokens_details": {
			"audio_tokens": 0,
			"cached_tokens": 0
		}
	}
}
```

は？？

よくよく確認してみると、`get_current_weather`の引数が`{\"location\":\"東京\"}`になっていた。。。`Tokyo`じゃないといけない。。。
再度、使うことができる関数の一覧を取得するところからやってみる

取得結果

```json
{
	"id": "chatcmpl-CDtk5JftDdfB5oeouX04Xe0VaLjNS",
	"choices": [
		{
			"finish_reason": "tool_calls",
			"index": 0,
			"logprobs": null,
			"message": {
				"content": null,
				"refusal": null,
				"role": "assistant",
				"annotations": [],
				"tool_calls": [
					{
						"id": "call_DsLAWKX5VLGouU9oz417Bgwd",
						"function": {
							"arguments": "{\"location\":\"Tokyo, Japan\"}",
							"name": "get_current_weather"
						},
						"type": "function"
					}
				]
			}
		}
	],
	"created": 1757428573,
	"model": "gpt-4o-2024-08-06",
	"object": "chat.completion",
	"service_tier": "default",
	"system_fingerprint": "fp_1827dd0c55",
	"usage": {
		"completion_tokens": 17,
		"prompt_tokens": 81,
		"total_tokens": 98,
		"completion_tokens_details": {
			"accepted_prediction_tokens": 0,
			"audio_tokens": 0,
			"reasoning_tokens": 0,
			"rejected_prediction_tokens": 0
		},
		"prompt_tokens_details": {
			"audio_tokens": 0,
			"cached_tokens": 0
		}
	}
}
```

Tokyo になってくれない（泣）
ただ、関数の内容をよく見ると`if "tokyo" in location.lower():`になっていたので`Tokyo, Japan`でも大丈夫そう

再度メッセージを作成し、Chat Completions API にリクエストをした結果

```json
{
	"id": "chatcmpl-CDtnH1ltu4LCkZ6yEn7SojNNzQGQe",
	"choices": [
		{
			"finish_reason": "stop",
			"index": 0,
			"logprobs": null,
			"message": {
				"content": "現在の東京の気温は10度です。",
				"refusal": null,
				"role": "assistant",
				"annotations": []
			}
		}
	],
	"created": 1757428771,
	"model": "gpt-4o-2024-08-06",
	"object": "chat.completion",
	"service_tier": "default",
	"system_fingerprint": "fp_cbf1785567",
	"usage": {
		"completion_tokens": 11,
		"prompt_tokens": 59,
		"total_tokens": 70,
		"completion_tokens_details": {
			"accepted_prediction_tokens": 0,
			"audio_tokens": 0,
			"reasoning_tokens": 0,
			"rejected_prediction_tokens": 0
		},
		"prompt_tokens_details": {
			"audio_tokens": 0,
			"cached_tokens": 0
		}
	}
}
```

めでたし、めでたし

## 第 3 章 プロンプトエンジニアリング

Zero-Shot プロンプティングとか Few-Shot の紹介

Zero-shot Chain-ofThought プロンプティング（Zero-shot CoT プロンプティング）:　 Zero-Shot に「ステップバイステップで考えてください」と最後に付け加える。

## 第 4 章 LangChain の基礎

LangChain: LLM アプリケーション開発のフレームワーク

LangChain は当初 langchain という一つのパッケージだったが機能追加を繰り返すことで依存関係などで問題が起きるようになった。
そのため languchain-core というパッケージを作成し、周辺機能は別パッケージで提供できるようにした。

OpenAI 用のインテグレーションである langchain-openai や Anthropic のインテグレーションである
languchain-anthropic などさまざまなインテグレーションがある。

LangSmith: LangChain の動作トレースを簡単に収集できる。デバックに役立つみたい

LangSmith に登録して API キーを発行してこんな感じで設定。

```python
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_ENDPOINT"] = "https://api.smith.langchain.com"
os.environ["LANGCHAIN_API_KEY"] = userdata.get("LANGCHAIN_API_KEY")
os.environ["LANGCHAIN_PROJECT"] = "agent-book"
```

### 4.2 LLM/Chat model

LangChain で OpenAI の Chat CompletionsAPI を使った例

```python
from langchain_core.messages import AIMessage, HumanMessage, SystemMessage
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)

messages = [
    SystemMessage("You are a helpful assistant."),
    HumanMessage("こんにちは！私はジョンと言います"),
    AIMessage(content="こんにちは、ジョンさん！どのようにお手伝いできますか？"),
    HumanMessage(content="私の名前がわかりますか？"),
]

ai_message = model.invoke(messages)
print(ai_message.content)
```

`messages`の中の対応関係はこんな感じ

- `SystemMessage`: `"role":"system"`
- `HumanMessage`: `"role":"user"`
- `AIMessage`:`"role":"assistant"`

出力

```text
はい、あなたの名前はジョンさんです。何か特別なことについてお話ししたいですか？
```

ストリーミングはこんな感じで実装

```python
from langchain_core.messages import SystemMessage, HumanMessage
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)

messages = [
    SystemMessage("You are a helpful assistant."),
    HumanMessage("こんにちは！"),
]

for chunk in model.stream(messages):
    print(chunk.content, end="", flush=True)
```

継承関係を簡単に書くと下記のようになっている。ChatOpenAI から Anthropic の Chat model である
langchain-antropic を利用したい場合、読み込みパッケージを変えるだけで行けるみたい

```txt
Runnable
 └─ BaseLanguageModel
    ├── BaseLLM
    |   ├── OpneAI
    |   └── その他LLM
    └── BaseChatModel
        ├── ChatOpenAI
        └── その他のChat Model
```

### 4.3 プロンプトテンプレート

プロンプトをテンプレート化しておくことで、ユーザが入力したキーワードから API に投げるプロンプトを生成する。

シンプルな例

```python
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate.from_template("""以下の料理のレシピを考えてください。

料理名: {dish}""")

prompt_value = prompt.invoke({"dish": "カレー"})
print(prompt_value.text)
```

こんなプロンプトになっている

```txt
以下の料理のレシピを考えてください。

料理名: カレー
```

MessagePlaceholder を使用するとチャット形式のモデルに会話履歴を含めることができる

```python
from langchain_core.messages import AIMessage, HumanMessage
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        MessagesPlaceholder("chat_history", optional=True),
        ("human", "{input}"),
    ]
)

prompt_value = prompt.invoke(
    {
        "chat_history": [
            HumanMessage(content="こんにちは！私はジョンと言います！"),
            AIMessage("こんにちは、ジョンさん！どのようにお手伝いできますか？"),
        ],
        "input": "私の名前が分かりますか？",
    }
)
print(prompt_value)
```

出力

```txt
messages=[SystemMessage(content='You are a helpful assistant.', additional_kwargs={}, response_metadata={}), HumanMessage(content='こんにちは！私はジョンと言います！', additional_kwargs={}, response_metadata={}), AIMessage(content='こんにちは、ジョンさん！どのようにお手伝いできますか？', additional_kwargs={}, response_metadata={}), HumanMessage(content='私の名前が分かりますか？', additional_kwargs={}, response_metadata={})]
```

LangSmith を利用するとソースコードとは別にプロンプトを管理するようなことができるみたい。

### 4.4 Output parser

Output parser を使用することで LLM からの出力をプログラム的に扱うことが可能。例えば特定の形式の JSON での返答を指定できる。

pydantic を使用してフィールドの定義を行う

```python
from pydantic import BaseModel, Field


class Recipe(BaseModel):
    ingredients: list[str] = Field(description="ingredients of the dish")
    steps: list[str] = Field(description="steps to make the dish")
```

作成したフィールド定義を langchain の output parser に指定する

```python
from langchain_core.output_parsers import PydanticOutputParser

output_parser = PydanticOutputParser(pydantic_object=Recipe)
format_instructions = output_parser.get_format_instructions()
print(format_instructions)
```

実際の中身を見てみるとプロンプトの内容で JSON の内容を指示しているような感じみたい

````text
The output should be formatted as a JSON instance that conforms to the JSON schema below.

As an example, for the schema {"properties": {"foo": {"title": "Foo", "description": "a list of strings", "type": "array", "items": {"type": "string"}}}, "required": ["foo"]}
the object {"foo": ["bar", "baz"]} is a well-formatted instance of the schema. The object {"properties": {"foo": ["bar", "baz"]}} is not well-formatted.

Here is the output schema:
　```
{"properties": {"ingredients": {"description": "ingredients of the dish", "items": {"type": "string"}, "title": "Ingredients", "type": "array"}, "steps": {"description": "steps to make the dish", "items": {"type": "string"}, "title": "Steps", "type": "array"}}, "required": ["ingredients", "steps"]}
　```
````

このような output parser を利用した ChatPromptTemplate を作成

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "ユーザーが入力した料理のレシピを考えてください。\n\n"
            "{format_instructions}",
        ),
        ("human", "{dish}"),
    ]
)

prompt_with_format_instructions = prompt.partial(
    format_instructions=format_instructions
)
```

プロンプトに対して入力を与えてどのようなプロンプトが作成されているのか見てみる

```python
prompt_value = prompt_with_format_instructions.invoke({"dish": "カレー"})
print("=== role: system ===")
print(prompt_value.messages[0].content)
print("=== role: user ===")
print(prompt_value.messages[1].content)
```

こんな感じの内容ができている

````text
=== role: system ===
ユーザーが入力した料理のレシピを考えてください。

The output should be formatted as a JSON instance that conforms to the JSON schema below.

As an example, for the schema {"properties": {"foo": {"title": "Foo", "description": "a list of strings", "type": "array", "items": {"type": "string"}}}, "required": ["foo"]}
the object {"foo": ["bar", "baz"]} is a well-formatted instance of the schema. The object {"properties": {"foo": ["bar", "baz"]}} is not well-formatted.

Here is the output schema:
　```
{"properties": {"ingredients": {"description": "ingredients of the dish", "items": {"type": "string"}, "title": "Ingredients", "type": "array"}, "steps": {"description": "steps to make the dish", "items": {"type": "string"}, "title": "Steps", "type": "array"}}, "required": ["ingredients", "steps"]}
　```
=== role: user ===
カレー
````

実際に LLM に投げてみる

```python
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)

ai_message = model.invoke(prompt_value)
print(ai_message.content)
```

出力

```json
{
	"ingredients": [
		"鶏肉 500g",
		"玉ねぎ 2個",
		"にんじん 1本",
		"じゃがいも 2個",
		"カレールー 1箱",
		"水 800ml",
		"サラダ油 大さじ2",
		"塩 適量",
		"こしょう 適量"
	],
	"steps": [
		"鶏肉は一口大に切り、塩とこしょうをふる。",
		"玉ねぎは薄切り、にんじんとじゃがいもは一口大に切る。",
		"鍋にサラダ油を熱し、玉ねぎを炒めて透明になるまで炒める。",
		"鶏肉を加え、表面が白くなるまで炒める。",
		"にんじんとじゃがいもを加え、全体を混ぜる。",
		"水を加え、煮立ったらアクを取り除く。",
		"弱火にして、蓋をして約20分煮る。",
		"カレールーを加え、溶かしながらさらに10分煮る。",
		"味を見て、必要に応じて塩で調整する。",
		"ご飯と一緒に盛り付けて完成。"
	]
}
```

ちゃんと材料とステップが順番になっている

もし、Pydanamic のインスタンスに変換して使用したい場合はこんな感じで変換可能

```python
recipe = output_parser.invoke(ai_message)
print(type(recipe))
print(recipe)
```

StrOutputParser なるものもあるみたい。必要性とかは後々説明があるみたい。

```python
from langchain_core.messages import AIMessage
from langchain_core.output_parsers import StrOutputParser

output_parser = StrOutputParser()

ai_message = AIMessage(content="こんにちは。私はAIアシスタントです。")
ai_message = output_parser.invoke(ai_message)
print(type(ai_message))
print(ai_message)
```

### 4.5 Chain - LangChain Expression Language (LCEL) の概要

- LangChain Expression Language (LCEL): LLM を使用した処理を連鎖的につなげる際に使用する
  LangChain での Chain の記述方法。プロンプトや LLM を「|」でつなげて使用する。

使用例：prompt と model をつないだ例。ChatPromptTemplate と ChatOpenAI を用意して、それをつないだ chain を作成

```py
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "ユーザーが入力した料理のレシピを考えてください。"),
        ("human", "{dish}"),
    ]
)

model = ChatOpenAI(model_name="gpt-4o-mini", temperature=0)

chain = prompt | model

ai_message = chain.invoke({"dish": "カレー"})
print(ai_message.content)
```

出力

```txt
カレーのレシピをご紹介します。シンプルで美味しい基本のカレーを作りましょう。

### 材料（4人分）
- 鶏肉（もも肉または胸肉）: 400g
- 玉ねぎ: 2個
- にんじん: 1本
- じゃがいも: 2個
- カレールー: 1箱（約200g）
- サラダ油: 大さじ2
- 水: 800ml
- 塩: 適量
- 胡椒: 適量
- お好みでガーリックパウダーや生姜: 適量

### 作り方
1. **材料の下ごしらえ**:
   - 鶏肉は一口大に切り、塩と胡椒をふっておきます。
   - 玉ねぎは薄切り、にんじんは輪切り、じゃがいもは一口大に切ります。

2. **炒める**:
   - 大きめの鍋にサラダ油を熱し、玉ねぎを中火で炒めます。玉ねぎが透明になるまで炒めます。
   - 鶏肉を加え、表面が白くなるまで炒めます。

3. **野菜を加える**:
   - にんじんとじゃがいもを鍋に加え、全体をよく混ぜます。

4. **煮る**:
   - 水を加え、強火で煮立たせます。煮立ったら、アクを取り除き、中火にして蓋をし、約15分煮ます。

5. **カレールーを加える**:
   - カレールーを割り入れ、よく溶かします。さらに10分ほど煮込み、全体がなじんだら火を止めます。

6. **味を調える**:
   - お好みで塩や胡椒で味を調整します。

7. **盛り付け**:
   - ご飯と一緒に盛り付けて、お好みで福神漬けやらっきょうを添えて完成です。

### おすすめのトッピング
- 煮卵
- チーズ
- ほうれん草のソテー

この基本のカレーは、具材を変えたり、スパイスを追加したりすることでアレンジが可能です。お好みのスタイルで楽しんでください！
```

StrOutputParser をさらに chain に追加するとそのまま文字列で結果を取得できる（何かしらの結果が変わってるわけではないっぽい）。

```py
from langchain_core.output_parsers import StrOutputParser

chain = prompt | model | StrOutputParser()
output = chain.invoke({"dish": "カレー"})
print(output)
```

PydanticOutputParser を利用してクラスのインスタンスに結果を格納する chain を作成する

Recip クラスを定義して

```py
from langchain_core.output_parsers import PydanticOutputParser
from pydantic import BaseModel, Field


class Recipe(BaseModel):
    ingredients: list[str] = Field(description="ingredients of the dish")
    steps: list[str] = Field(description="steps to make the dish")


output_parser = PydanticOutputParser(pydantic_object=Recipe)
```

```py
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "ユーザーが入力した料理のレシピを考えてください。\n\n{format_instructions}"),
        ("human", "{dish}"),
    ]
)

prompt_with_format_instructions = prompt.partial(
    format_instructions=output_parser.get_format_instructions()
)

model = ChatOpenAI(model="gpt-4o-mini", temperature=0).bind(
    response_format={"type": "json_object"}
)
```

chain でつなげる要素の準備と chain の作成。prompt と model を準備している。
prompt の{format_instructions}の部分に先ほど作成した Recipe クラスの内容で返すように要求するプロンプトが格納される。

```py
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "ユーザーが入力した料理のレシピを考えてください。\n\n{format_instructions}"),
        ("human", "{dish}"),
    ]
)

prompt_with_format_instructions = prompt.partial(
    format_instructions=output_parser.get_format_instructions()
)

model = ChatOpenAI(model="gpt-4o-mini", temperature=0).bind(
    response_format={"type": "json_object"}
)

chain = prompt_with_format_instructions | model | output_parser
```

実行

```py
recipe = chain.invoke({"dish": "カレー"})
print(type(recipe))
print(recipe)
```

出力

```py
<class '__main__.Recipe'>
ingredients=['鶏肉 500g', '玉ねぎ 2個', 'にんじん 1本', 'じゃがいも 2個', 'カレールー 1箱', '水 800ml', 'サラダ油 大さじ2', '塩 適量', 'こしょう 適量'] steps=['鶏肉は一口大に切り、塩とこしょうをふる。', '玉ねぎは薄切り、にんじんとじゃがいもは一口大に切る。', '鍋にサラダ油を熱し、玉ねぎを炒めて透明になるまで炒める。', '鶏肉を加え、表面が白くなるまで炒める。', 'にんじんとじゃがいもを加え、全体を混ぜる。', '水を加え、沸騰したらアクを取り、弱火で20分煮る。', 'カレールーを加え、よく溶かしてさらに10分煮る。', '味を見て、必要に応じて塩で調整する。', 'ご飯と一緒に盛り付けて完成。']
```

このようにして Recipe クラスのインスタンスを得ることができた。つまり、`chain.invoke`の部分でプロンプトの穴埋め・LLM の呼び出し・出力の変換が連鎖的に行われてたことがわかる。

また、モデルに制約はあるが`with_structured_output`をしようすることで記述を簡略化できる。

```py
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from pydantic import BaseModel, Field


class Recipe(BaseModel):
    ingredients: list[str] = Field(description="ingredients of the dish")
    steps: list[str] = Field(description="steps to make the dish")


prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "ユーザーが入力した料理のレシピを考えてください。"),
        ("human", "{dish}"),
    ]
)

model = ChatOpenAI(model="gpt-4o-mini")

chain = prompt | model.with_structured_output(Recipe)

recipe = chain.invoke({"dish": "カレー"})
print(type(recipe))
print(recipe)
```

### 4.6 LangChain の RAG に関するコンポーネント

- Document loader: データソースからドキュメントを読み込む
- Document transformer: ドキュメントに何らかの変換をかける
- Embedding model: ドキュメントをベクトル化する
- Vector store: ベクトル化したドキュメントの保存先
- Retriever: 入力したテキストと関連するドキュメントを検索する

ここまで、書籍の案内通りに google colab を使用していたがなんか使いにくかったので普通にローカルでの実行に変更。こまったら戻る。。。

LangChain では様々な Document loader が用意されている。S3 用とか Google Drive とか Web ページとか。
今回は GitLoader を使用する。

```py
from langchain_community.document_loaders import GitLoader
from langchain_openai import ChatOpenAI
from langchain_core.messages import AIMessage, HumanMessage, SystemMessage
from dotenv import load_dotenv

load_dotenv()


def file_filter(file_path: str) -> bool:
    return file_path.endswith(".mdx")


loader = GitLoader(
    clone_url="https://github.com/langchain-ai/langchain",
    repo_path="./langchain",
    branch="master",
    file_filter=file_filter,
)

raw_docs = loader.load()
print(len(raw_docs))
```

実行結果

```txt
442
```

このコードでロカールに langchain フォルダが作成されて、そこにレポジトリが clone される。
その clone した内容から mdx ファイル（442 ファイル）だけを`row_docs`の内容として読み込んでいる。

次に、Document transformer を使用して読み込んだドキュメントをチャンクに分割する。
LangChain ではドキュメントをチャンクかする機能群は「Text splitter」と呼ばれていて`langchain-text-splitters`というパッケージに分離されている。

今回は CharacterTextSplitter を使用してチャンク分けを行う。他にも HTML をプレーンテキストにするものやメタデータを抽出するものや翻訳するものなどさまざまなものがある。

```py
from langchain_text_splitters import CharacterTextSplitter

text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)

docs = text_splitter.split_documents(raw_docs)
print(len(docs))
```

実行結果はこのような感じになる。

```txt
Created a chunk of size 6803, which is longer than the specified 1000
Created a chunk of size 3302, which is longer than the specified 1000
...
Created a chunk of size 1443, which is longer than the specified 1000
Created a chunk of size 3138, which is longer than the specified 1000
Created a chunk of size 1100, which is longer than the specified 1000
1573
```

チャンク数は 1573 になる。442 ファイルの内容が 1573 のチャンクに分割されたということである。

`Created a chunk of size XXXX, which is longer than the specified 1000`とあるが、これは`CharacterTextSplitter`がデフォルトで段落毎にチャンクを作成しているからみたい。
つまり、一つの段落の文字数が多すぎて指定しているチャンクサイズを超えてしまったことをログとして出している。

ん？段落毎にチャンク分ける使用ならチャンクサイズの指定はいらないのでは？

ということでもないみたい。段落で区切ったうえで小さなチャンクは一つのチャンクにまとめられるらしい。しかし、大きすぎる段落の場合はそれで一つのチャンクになる仕様らしい。
つまり、チャンクサイズの指定を大きくするとチャンク数は減る。実際にチャンクサイズを 5000 にしてみるとチャンク数は 586 まで減った。
今回は機械的にチャンクを分けているが、このチャンク分けを AI を使用してすることもできる。その場合は、意味合いを理解したチャンク分けのようなことができる。

チャンク分けなどの変換処理が終わるとそれをベクトル化する。
例えば、`text-embedding-3-small`を使用してベクトル化するコードは下記のものになる。実行すると約 1500 次元のベクトルが出力される。

```py
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

query = "AWSのS3からデータを読み込むためのDocument loaderはありますか？"

vector = embeddings.embed_query(query)
print(len(vector))
print(vector)
```

先ほど、ベクトル化したデータを Vector store に格納する。今回はローカルでも使用可能な Chroma というパッケージを使用。
LangChain において関連するドキュメントを得るインターフェイスを「Retriever」という。

この例は、チャンク化したコードをベクトル化して DB に格納して query で検索する例

```py
from langchain_chroma import Chroma

db = Chroma.from_documents(docs, embeddings)

retriever = db.as_retriever()

query = "AWSのS3からデータを読み込むためのDocument loaderはありますか？"

context_docs = retriever.invoke(query)
print(f"len = {len(context_docs)}")

first_doc = context_docs[0]
print(f"metadata = {first_doc.metadata}")
print(first_doc.page_content)
```

結果

````md
len = 4
metadata = {'file_name': 'aws.mdx', 'file_path': 'docs/docs/integrations/providers/aws.mdx', 'file_type': '.mdx', 'source': 'docs/docs/integrations/providers/aws.mdx'}

### AWS S3 Directory and File

> [Amazon Simple Storage Service (Amazon S3)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-folders.html)
> is an object storage service.
> [AWS S3 Directory](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-folders.html) >[AWS S3 Buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBucket.html)

See a [usage example for S3DirectoryLoader](/docs/integrations/document_loaders/aws_s3_directory).

See a [usage example for S3FileLoader](/docs/integrations/document_loaders/aws_s3_file).

```python
from langchain_community.document_loaders import S3DirectoryLoader, S3FileLoader
```

### Amazon Textract

> [Amazon Textract](https://docs.aws.amazon.com/managedservices/latest/userguide/textract.html) is a machine
> learning (ML) service that automatically extracts text, handwriting, and data from scanned documents.

See a [usage example](/docs/integrations/document_loaders/amazon_textract).
````

なんか結果は取れてるっぽい。

ベクトル化してそれを検索できてそうなので、ここから Chain として実装をしていく

まずはプロンプトテンプレートの作成

```py
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_template('''\
以下の文脈だけを踏まえて質問に回答してください。

文脈: """
{context}
"""

質問: {question}
''')

model = ChatOpenAI(model_name="gpt-4o-mini", temperature=0)
```

そして、chain の実装

```py
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

output = chain.invoke(query)
print(output)
```

わかんね。ということでちょっと調べる。

- そもそも`chain`にかく要素は全て`invoke`メソッドを持つものみたい。chain はそれを順番に実行してくれる。
- ただし、辞書型のものを`chain`に入れた場合、キーがテンプレート変数名、値が`invoke`可能オブジェクトとなる。
- `RunnablePassthrough()`は`chain.invoke(query)`の引数の内容をそのまま返すだけの`invoke`をし、その後の処理で引数の`query`の内容を`question`として扱うことを可能にしている。
- `chain`の先頭の要素は実行時の引数を使用した`invoke`メソッドを実行し、それ以降は前要素の`invoke`の結果を引数に取り自身の`invoke`の処理を行う

出力結果

```txt
はい、AWSのS3からデータを読み込むためのDocument loaderとして、`S3DirectoryLoader`と`S3FileLoader`があります。これらは、AWS S3のディレクトリやファイルからデータを読み込むために使用されます。
```

<details close markdown="1">
  <summary>プログラム全文</summary>

```py
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import OpenAIEmbeddings
from langchain_chroma import Chroma
from langchain_text_splitters import CharacterTextSplitter
from langchain_community.document_loaders import GitLoader
from dotenv import load_dotenv

load_dotenv()


def file_filter(file_path: str) -> bool:
    return file_path.endswith(".mdx")


loader = GitLoader(
    clone_url="https://github.com/langchain-ai/langchain",
    repo_path="./langchain",
    branch="master",
    file_filter=file_filter,
)

raw_docs = loader.load()
print(f"{len(raw_docs)=}")


text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)

docs = text_splitter.split_documents(raw_docs)
print(f"{len(docs)=}")


embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

db = Chroma.from_documents(docs, embeddings)

retriever = db.as_retriever()

query = "AWSのS3からデータを読み込むためのDocument loaderはありますか？"

prompt = ChatPromptTemplate.from_template('''\
以下の文脈だけを踏まえて質問に回答してください。

文脈: """
{context}
"""

質問: {question}
''')

model = ChatOpenAI(model_name="gpt-4o-mini", temperature=0)


chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

output = chain.invoke(query)
print(output)
```

</details>

## 第 5 章 LangChain Expression Language(LCEL)徹底解説

### Runnable と RunnableSequence - LCEL の最も基本的な構成

LCEL の最も基本的な実装は Prompt template・Chat model・Output parser の三つの連結

例えば下記のコードの`StrOutputParsere`, `ChatPromptTemplate`, `ChatOpenAI`は LangChain の`Runnable`という抽象基底クラスを継承している。
「chain」はこの`Runnable`を「|」でつないだもの( = `RunnableSequence`)。

```py
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "ユーザーが入力した料理のレシピを考えてください。"),
        ("human", "{dish}"),
    ]
)

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)

output_parser = StrOutputParser()

chain = prompt | model | output_parser

output = chain.invoke({"dish": "カレー"})
print(output)
```

ストリーミングで実行したい場合は`stream`メソッドを使用

```py
chain = prompt | model | output_parser

for chunk in chain.stream({"dish": "カレー"}):
    print(chunk, end="", flush=True)
```

複数入力を一度にしたい場合は`batch`を使用

```py
chain = prompt | model | output_parser

outputs = chain.batch([{"dish": "カレー"}, {"dish": "うどん"}])
print(outputs)
```

バッチのときの出力結果はこんな感じ

```txt
['カレーのレシピをご紹介します。シンプルで美味しい基本のカレーを作りましょう。\n\n### 材料（4人分）\n- 鶏肉（もも肉または胸肉）: 400g\n- 玉ねぎ: 2個\n- にんじん: 1本\n- じゃがいも: 2個\n- カレールー: 1箱（約200g）\n- サラダ油: 大さじ2\n- 水: 800ml\n- 塩: 適量\n- 胡椒: 適量\n- お好みでガーリックパウダーや生姜: 適量\n\n### 作り方\n1. **材料の下ごしらえ**:\n   - 鶏肉は一口大に切り、塩と胡椒を振っておきます。\n   - 玉ねぎは薄切り、にんじんは輪切り、じゃがいもは一口大に切ります。\n\n2. **炒める**:\n   - 大きめの鍋にサラダ油を熱し、玉ねぎを中火で炒めます。玉ねぎが透明になるまで炒めます。\n   - 鶏肉を加え、表面が白くなるまで炒めます。\n\n3. **野菜を加える**:\n   - にんじんとじゃがいもを鍋に加え、全体をよく混ぜます。\n\n4. **煮る**:\n   - 水を加え、強火で煮立たせます。煮立ったら、アクを取り除き、中火にして蓋をし、約15分煮ます。\n\n5. **カレールーを加える**:\n   - カレールーを割り入れ、よく溶かします。さらに10分ほど煮込み、全体がなじんだら火を止めます。\n\n6. **味を調える**:\n   - お好みで塩や胡椒で味を調整します。\n\n7. **盛り付け**:\n   - ご飯と一緒に盛り付けて、お好みで福神漬けやらっきょうを添えて完成です。\n\n### おすすめのトッピング\n- 煮卵\n- チーズ\n- ほうれん草のソテー\n\nこの基本のカレーはアレンジがしやすいので、野菜や肉を変えて自分好みのカレーを楽しんでください！', 'うどんのレシピをご紹介します。シンプルで美味しい「かけうどん」の作り方です。\n\n### 材料（2人分）\n- うどん（乾燥または生）: 2玉\n- だし汁: 600ml\n  - だしの素（または昆布と鰹節）: 適量\n- 醤油: 大さじ2\n- みりん: 大さじ1\n- 塩: 少々\n- トッピング（お好みで）:\n  - ネギ（小口切り）\n  - 天かす\n  - かまぼこ\n  - ほうれん草やわかめ\n  - 生卵\n\n### 作り方\n1. **だし汁を作る**:\n   - 鍋に水600mlを入れ、だしの素を加えて中火にかけます。昆布と鰹節を使う場合は、昆布を水に浸けておき、沸騰直前に取り出し、鰹節を加えて数分煮出します。その後、こしてだし汁を作ります。\n\n2. **うどんを茹でる**:\n   - 別の鍋にたっぷりの水を沸かし、うどんをパッケージの指示に従って茹でます。茹で上がったら、冷水でしっかりと洗い、ぬめりを取ります。\n\n3. **だし汁を味付けする**:\n   - 作っただし汁に醤油、みりん、塩を加えて味を調えます。軽く煮立たせて、味がなじむようにします。\n\n4. **盛り付け**:\n   - 茹でたうどんを器に盛り、熱々のだし汁をかけます。お好みのトッピング（ネギ、天かす、かまぼこ、ほうれん草など）をのせます。\n\n5. **仕上げ**:\n   - お好みで生卵をトッピングしたり、さらに香りを楽しむためにごまを振りかけても良いでしょう。\n\n### 提供\n熱々のかけうどんを楽しんでください！お好みで七味唐辛子をかけても美味しいです。']
```

また、chain 自体も`Runnable`クラスになるので chain 同士の結合も可能

```py
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv

load_dotenv()

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)

output_parser = StrOutputParser()

cot_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "ユーザーの質問にステップバイステップで回答してください。"),
        ("human", "{question}"),
    ]
)

cot_chain = cot_prompt | model | output_parser

summarize_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "ステップバイステップで考えた回答から結論だけ抽出してください。"),
        ("human", "{text}"),
    ]
)

summarize_chain = summarize_prompt | model | output_parser

cot_summarize_chain = cot_chain | summarize_chain
output = cot_summarize_chain.invoke({"question": "10 + 2 * 3"})
print(output)
```

出力結果

```txt
10 + 2 * 3 の答えは **16** です。
```

`("human", "{text}"),`この部分で`text`というキーワードを適当な文字列に変えても同じような結果が得られた。引数が一つなら勝手に埋めてくれるみたい。
複数の場合だと`RunnablePassthrough()`とかうまいことつかうのかな？

LangSmith をつかうとこんな感じで Chain 内聞を確認できる

![image]({{site.baseurl}}/images/reading/rag_aiagent_nyuumon/05_langsmith.png)

### 5.2 RunnableLambda - 任意の関数を Runnable にする

任意の関数を Runnable にして chain に渡す例：

```py
from langchain_core.runnables import RunnableLambda
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv

load_dotenv()
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        ("human", "{input}"),
    ]
)

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)

output_parser = StrOutputParser()


def upper(text: str) -> str:
    return text.upper()


chain = prompt | model | output_parser | RunnableLambda(upper)

ai_message = chain.invoke({"input": "Hello!"})
print(ai_message)
```

出力

```txt
HELLO! HOW CAN I ASSIST YOU TODAY?
```

出力が全て大文字になっていることがわかる。
また、chain デコレータを使用することも可能

```py
from langchain_core.runnables import chain


@chain
def upper(text: str) -> str:
    return text.upper()


chain = prompt | model | output_parser | upper

ai_message = chain.invoke({"input": "Hello!"})
 print(ai_message)
```

#### RunnableLambda への自動変換

別に`RunnableLambda`なくてもいけるみたい。なんやねんそれ。。。(笑)

```py
def upper(text: str) -> str:
    return text.upper()


chain = prompt | model | output_parser | upper
```

Runnable の入力と出力の型には注意を払う必要あり。

```py
def upper(text: str) -> str:
    return text.upper()


# これはダメ
# 以下のコードを実行するとエラーになります
# output = chain.invoke({"input": "Hello!"})
chain = prompt | model | upper

# これはOK
chain = prompt | model | StrOutputParser() | upper

```

`model`は`AImessage`を出力するが、`upper`は`str`を引数としているため、エラーになる。間に`StrOutputParser()`を入れる必要がある。

### 5.3 RunnableParallel - 複数の Runnable を並列につなげる

Runnable を並列につなげることができる。

楽観的な意見を生成する Chain と悲観的な意見を生成する Chain を並列で実装する例

```py
from langchain_core.runnables import RunnableParallel
import pprint
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv

load_dotenv()

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)
output_parser = StrOutputParser()

optimistic_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "あなたは楽観主義者です。ユーザーの入力に対して楽観的な意見をください。"),
        ("human", "{topic}"),
    ]
)
optimistic_chain = optimistic_prompt | model | output_parser

pessimistic_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "あなたは悲観主義者です。ユーザーの入力に対して悲観的な意見をください。"),
        ("human", "{topic}"),
    ]
)
pessimistic_chain = pessimistic_prompt | model | output_parser


parallel_chain = RunnableParallel(
    {
        "optimistic_opinion": optimistic_chain,
        "pessimistic_opinion": pessimistic_chain,
    }
)

output = parallel_chain.invoke({"topic": "生成AIの進化について"})
pprint.pprint(output)
```

出力

```txt
{'optimistic_opinion': '生成AIの進化は本当に素晴らしいですね！技術が進むことで、私たちの生活がより便利で豊かになる可能性が広がっています。クリエイティブな作業や問題解決の手助けをしてくれるAIが増えてきて、私たちのアイデアを実現する手助けをしてくれるのです。これからも新しい発見や革新が続くでしょうし、私たちの未来はますます明るいものになると信じています！',
 'pessimistic_opinion': '生成AIの進化は確かに目覚ましいものがありますが、その一方で多くの懸念も抱えています。技術が進化することで、私たちの仕事が奪われたり、情報の信頼性が低下したりするリスクが高まっています。さらに、AIが生成するコンテンツが人間の創造性を脅かし、私たちの思考や感情に悪影響を及ぼす可能性もあります。結局のところ、便利さの裏には常にリスクが潜んでいるのです。私たちがこの技術をどのように扱うかが、未来における大きな課題となるでしょう。'}
```

楽観的な意見と悲観的な意見が dict になって出力される。

そして、その並列で出した意見をまとめるようなことものできる。

楽観的な意見と悲観的な意見を客観的意見としてまとめる Chain を動かす例。

最後の方に意見をまとめる用の Template を作成する

```py
from langchain_core.runnables import RunnableParallel
import pprint
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv

load_dotenv()

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)
output_parser = StrOutputParser()

optimistic_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "あなたは楽観主義者です。ユーザーの入力に対して楽観的な意見をください。"),
        ("human", "{topic}"),
    ]
)
optimistic_chain = optimistic_prompt | model | output_parser

pessimistic_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "あなたは悲観主義者です。ユーザーの入力に対して悲観的な意見をください。"),
        ("human", "{topic}"),
    ]
)
pessimistic_chain = pessimistic_prompt | model | output_parser


parallel_chain = RunnableParallel(
    {
        "optimistic_opinion": optimistic_chain,
        "pessimistic_opinion": pessimistic_chain,
    }
)

synthesize_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "あなたは客観的AIです。2つの意見をまとめてください。"),
        ("human", "楽観的意見: {optimistic_opinion}\n悲観的意見: {pessimistic_opinion}"),
    ]
)

synthesize_chain = (
    RunnableParallel(
        {
            "optimistic_opinion": optimistic_chain,
            "pessimistic_opinion": pessimistic_chain,
        }
    )
    | synthesize_prompt
    | model
    | output_parser
)

output = synthesize_chain.invoke({"topic": "生成AIの進化について"})
print(output)
```

出力

```txt
生成AIの進化については、楽観的な意見と悲観的な意見が存在します。楽観的な見方では、生成AIの技術が進むことで私たちの生活が便利で豊かになり、クリエイティブな作業や問題解決のパートナーとしての役割を果たすことが期待されています。この進化により、新しい発見や革新が続き、未来が明るくなる可能性があるとされています。

一方で、悲観的な見方では、生成AIの進化には多くの懸念が伴い、仕事の喪失や情報の信頼性の低下といったリスクが高まると指摘されています。また、AIが生成するコンテンツが人間の創造性を脅かし、思考や感情に悪影響を及ぼす可能性も懸念されています。このように、便利さの裏には不安や危険が潜んでおり、技術の進化が本当に私たちの生活を豊かにするのか疑問を抱く声もあります。

総じて、生成AIの進化は期待と懸念の両方を抱えており、今後の展開に注目が必要です。
```

`RunnableParallel`はキーが str で値が Runnable だと勝手に RunnableParallel に変換してくれるみたい

```py
synthesize_chain = (
    # RunnableParallelは書かなくてもOK
    # RunnableParallel(
    #     {
    #         "optimistic_opinion": optimistic_chain,
    #         "pessimistic_opinion": pessimistic_chain,
    #     }

    # これでもOK
    {
        "optimistic_opinion": optimistic_chain,
        "pessimistic_opinion": pessimistic_chain,
    }
    | synthesize_prompt
    | model
    | output_parser
)
```

最後の客観的な意見をまとめる際に、最初に何の topic について質問をしたのかはこのプロンプトからはわからない。
それを伝える方法として itemgetter というものがある。

```py
synthesize_chain = (
    {
        "optimistic_opinion": optimistic_chain,
        "pessimistic_opinion": pessimistic_chain,
        "topic": itemgetter("topic"),
    }
    | synthesize_prompt
    | model
    | output_parser
)

output = synthesize_chain.invoke({"topic": "生成AIの進化について"})
print(output)
```

出力

```txt
生成AIの進化についての意見をまとめると、以下のようになります。

**楽観的意見**: 生成AIの進化は、私たちの生活を便利で豊かにする可能性を秘めています。AIはクリエイティブな作業や問題解決のサポートを行い、私たちのアイデアを実現するパートナーとして機能しています。また、教育、医療、エンターテインメントなどの分野での革新を促進し、より多くの人々が新しい知識や体験にアクセスできるようになることで、社会全体が進化することが期待されています。将来的には、AIと人間が協力してより良い世界を築く姿が見られるでしょう。

**悲観的意見**: 一方で、生成AIの進化には多くの懸念も伴います。技術の進化により、仕事が奪われるリスクや情報の信頼性が低下する可能性が高まっています。AIが生成するコンテンツが氾濫することで、真実と虚偽の区別が難しくなり、社会全体が混乱する恐れもあります。便利さの裏には常に危険が潜んでおり、慎重な対応が求められます。
```

### 5.4 RunnablePassthrough - 入力をそのまま出力する

RunnablePassthrough というものを使うと RunnableParalell を使用する際にその一部の入力をそのまま出力できる

Tavily という LLM や RAG 向けに最適化された検索エンジンを使用する。
ここでいう LLM や RAG 向けというのは、Google や Bing のように人向けではなく、AI アプリが使いやす用に設計されているという意味。
詳しくはわからん。。。（笑）

使用例

```py
from langchain_core.runnables import RunnablePassthrough
from langchain_community.retrievers import TavilySearchAPIRetriever
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from dotenv import load_dotenv

load_dotenv()

prompt = ChatPromptTemplate.from_template('''\
以下の文脈だけを踏まえて質問に回答してください。

文脈: """
{context}
"""

質問: {question}
''')

model = ChatOpenAI(model_name="gpt-4o-mini", temperature=0)

# kは検索件数
retriever = TavilySearchAPIRetriever(k=3)

chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

output = chain.invoke("東京の今日の天気は？")
print(output)
```

Tavily から取得した情報を検索して出している。4 章でやった構成に似ているが今回は Tavily を使用している。

Tavily での検索内容をどこで指定しているのかよくわからなかった（という疑問を 4 章でも持って解決したのを忘れていた。。。泣）。
イメージ的にはこんな感じになっており、呼び出した際の引数が自動で入るようになっている。

```py
{
  "context": retriever.invoke("東京の今日の天気は？"),
  "question": "東京の今日の天気は？"
}
```

retriever の検索結果も Chain 全体の出力に含めたい場合は RunnablePassthrough の assign というクラスメソッドを使用できる

chain 部分をこのように変更する

```py
chain = {
    "question": RunnablePassthrough(),
    "context": retriever,
} | RunnablePassthrough.assign(answer=prompt | model | StrOutputParser())

output = chain.invoke("東京の今日の天気は？")
pprint.pprint(output)
```

出力

```txt
{'answer': '東京の今日、2025年9月24日（水）の天気は「晴時々曇」となっています。',
 'context': [Document(metadata={'title': '東京（東京）の天気 - Yahoo!天気・災害 - Yahoo! JAPAN', 'source': 'https://weather.yahoo.co.jp/weather/jp/13/4410.html', 'score': 0.78886026, 'images': []}, page_content='現在位置： 天気・災害トップ > 関東・信越 > 東京都 > 東京（東京） 2025年9月24日 20時00分発表 * 9月24日(水) * 9月25日(木) 2025年9月24日 20時00分発表 | 日付 | 9月26日 (金) | 9月27日(土) | 9月28日(日) | 9月29日 (月) | 9月30日 (火) | 10月1日 (水) | | 天気 | 晴時々曇 | 曇時々晴 | 曇時々晴 | 曇時々晴 | 曇時々晴 | 曇り | 2025年9月24日 22時00分 発表 :   (C) Mapbox (C) OpenStreetMap (C) LY Corporation Yahoo! ### *東京都*に関する話題のワード :   (C) Mapbox (C) OpenStreetMap (C) LY Corporation Yahoo! 再生する9/24(水)18時\u3000北日本は雨風強まり荒れた天気\u3000九州南部も明け方にかけて土砂災害など警戒 プライバシーポリシー - プライバシーセンター - 利用規約 - ご意見・ご要望 - 広告掲載について - ヘルプ・お問い合わせ Copyright (C) 2025 Weather Map Co., Ltd. All Rights Reserved. © LY Corporation'),
             Document(metadata={'title': '東京都の天気 - Yahoo!天気・災害', 'source': 'https://weather.yahoo.co.jp/weather/jp/13/', 'score': 0.78054583, 'images': []}, page_content='パーソナル天気 現在位置： 天気・災害トップ > 関東・信越 > 東京都 |  | **九州北部に大雨\u3000最新情報まとめ**  * ・雨雲レーダー * ・警報・注意報 * ・大雨警戒レベルマップ * ・運行情報 * ・大雨に備える | |  | **全国の熱中症情報 危険な時間帯を確認して対策を** - 熱中症に備える保険 | # 東京都の天気 8月11日 22時00分発表 * *11*(月) * *12*(火) * *13*(水) * *14*(木) * *15*(金) * *16*(土) * *17*(日) * *18*(月) 関東・信越 * 東京 * 大島 * 八丈島 * 父島 最近見たエリア ## 都道府県概況 前線が、黄海付近から東日本の日本海側を通って、日本の東にのびています。   東京地方は、曇りや雨となっています。   11日は、前線や湿った空気の影響を受ける見込みです。曇り時々雨で、雷を伴う所があるでしょう。伊豆諸島では、雨や雷雨となる所がある見込みです。   12日は、前線や湿った空気の影響を受ける見込みです。このため、曇り時々雨で、夜のはじめ頃まで雷を伴う所があるでしょう。伊豆諸島では雨で雷を伴う所がある見込みです。   【関東甲信地方】関東甲信地方は、曇りや雨となっています。   11日は、前線や湿った空気の影響を受ける見込みです。このため、曇りや雨で、雷を伴い激しく降る所があるでしょう。   12日は、前線や湿った空気の影響を受ける見込みです。このため、曇りや雨で、雷を伴い激しく降る所があるでしょう。   関東地方と伊豆諸島の海上では、11日から12日にかけて、波が高い見込みです。12日はうねりを伴うでしょう。また、所々で霧が発生する見込みです。船舶は高波や視程障害に注意してください。 続きを見る * 北海道 :   + 道北 + 道央 + 道東 + 道南 * 東北 :   + 青森 + 岩手 + 宮城 + 秋田 + 山形 + 福島 * 関東・信越 :   + 東京 + 神奈川 + 埼玉 + 千葉 + 茨城 :   (C) Mapbox (C) OpenStreetMap (C) LY Corporation 再生する8/11(月)19時\u3000梅雨末期のような大雨\u3000警戒続く\u3000長崎県に線状降水帯のおそれ'),
             Document(metadata={'title': '東京の天気・気温：今日・明日と14日間(2週間)の1時間ごと ...', 'source': 'https://www.toshin.com/weather/detail?id=56682', 'score': 0.75852895, 'images': []}, page_content='* æ\x8e¡ç\x94¨æ\x83\x85å\xa0± 9æ\x9c\x8827æ\x97¥(å\x9c\x9f) 9æ\x9c\x8828æ\x97¥(æ\x97¥) 9æ\x9c\x8829æ\x97¥(æ\x9c\x88) 9æ\x9c\x8830æ\x97¥(ç\x81«) 10æ\x9c\x881æ\x97¥(æ°´) 10æ\x9c\x881æ\x97¥(æ°´) 10æ\x9c\x882æ\x97¥(æ\x9c¨) 10æ\x9c\x882æ\x97¥(æ\x9c¨) 10æ\x9c\x883æ\x97¥(é\x87\x91) 10æ\x9c\x883æ\x97¥(é\x87\x91) 10æ\x9c\x884æ\x97¥(å\x9c\x9f) 10æ\x9c\x884æ\x97¥(å\x9c\x9f) 10æ\x9c\x885æ\x97¥(æ\x97¥) 10æ\x9c\x885æ\x97¥(æ\x97¥) 10æ\x9c\x886æ\x97¥(æ\x9c\x88) 10æ\x9c\x886æ\x97¥(æ\x9c\x88) 10æ\x9c\x887æ\x97¥(ç\x81«) 10æ\x9c\x887æ\x97¥(ç\x81«) 10æ\x9c\x888æ\x97¥(æ°´) 10æ\x9c\x888æ\x97¥(æ°´) 10æ\x9c\x889æ\x97¥(æ\x9c¨) 10æ\x9c\x889æ\x97¥(æ\x9c¨) 10æ\x9c\x8810æ\x97¥(é\x87\x91) | å\x8c\x97 | å\x8c\x97 | å\x8c\x97æ\x9d± | æ\x9d± | æ\x9d±å\x8d\x97 | æ\x9d±å\x8d\x97 | å\x8d\x97 | å\x8d\x97 | å\x8d\x97 | æ\x9d±å\x8d\x97 | æ\x9d±å\x8d\x97 | å\x8d\x97 | å\x8d\x97 | æ\x9d±å\x8d\x97 | æ\x9d± | | å\x8c\x97è¥¿ | å\x8c\x97è¥¿ | å\x8c\x97è¥¿ | å\x8c\x97è¥¿ | å\x8c\x97è¥¿ | å\x8c\x97è¥¿ | å\x8c\x97 | å\x8c\x97 | å\x8c\x97 | å\x8c\x97æ\x9d± | å\x8c\x97æ\x9d± | æ\x9d± | æ\x9d±å\x8d\x97 | æ\x9d±å\x8d\x97 | æ\x9d±å\x8d\x97 | æ\x9d±å\x8d\x97 | æ\x9d±å\x8d\x97 | æ\x9d±å\x8d\x97 | æ\x9d±å\x8d\x97 | æ\x9d± | æ\x9d± | æ\x9d± | å\x8c\x97æ\x9d± | å\x8c\x97 |')],
 'question': '東京の今日の天気は？'}
```

retriever からのデータ（Tavily からの検索結果）も出力されていることがわかる。思いっきり文字化けしてるけどね。

## 第 6 章 Advanced RAG

このコードを基準としていろいろ手を加えていく。やっていることは 4 章でやったこととほぼ同じ

```py
from langchain_openai import ChatOpenAI
from langchain_core.runnables import RunnablePassthrough
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import OpenAIEmbeddings
from langchain_chroma import Chroma
from langchain_community.document_loaders import GitLoader
from langchain_text_splitters import CharacterTextSplitter
from dotenv import load_dotenv

load_dotenv()


def file_filter(file_path: str) -> bool:
    return file_path.endswith(".mdx")


loader = GitLoader(
    clone_url="https://github.com/langchain-ai/langchain",
    repo_path="./langchain",
    branch="master",
    file_filter=file_filter,
)

documents = loader.load()

text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

print(len(docs))

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
db = Chroma.from_documents(docs, embeddings)


prompt = ChatPromptTemplate.from_template('''\
以下の文脈だけを踏まえて質問に回答してください。

文脈: """
{context}
"""

質問: {question}
''')

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)

retriever = db.as_retriever()

chain = {
    "question": RunnablePassthrough(),
    "context": retriever,
} | prompt | model | StrOutputParser()

result = chain.invoke("LangChainの概要を教えて")

print(result)
```

出力

```txt
LangChainは、AIアプリケーションにおいて中心的なコンポーネントのための共通インターフェースを提供するフレームワークです。これにより、異なるプロバイダー特有の機能（例えば、ツール呼び出しや構造化出力）をサポートしつつ、チャットモデルなどのコンポーネントと標準的に対話することが可能になります。また、LangChainはカスタムチェーンを作成するための「LangChain Expression Language」（LCEL）を提供しており、これによりユーザーは柔軟なチェーンを構築できます。さらに、LangChainの概念ガイドは、フレームワークの用語や概念をより抽象的に説明し、ユーザーが深い理解を得る手助けをします。
```

### 6.3 検索クエリの工夫

#### HyDE (Hypothetical Document Embeddings)

シンプルな RAG の構成ではユーザーの質問に対してベクトル類似度の高いドキュメントを検索する。
しかし、本当に必要なのは質問に対する回答と類似するドキュメントである。
そこで、HyDE という手法がある。これは LLM に仮説的な回答を推論させて、その出力を類似度検索に使用する。

このように、仮説を出力する用の chain である`hypothetical_chain`を作成する。
そして、その結果を`retriever`に渡して最終的な結果を出力する

```py
hypothetical_prompt = ChatPromptTemplate.from_template("""\
次の質問に回答する一文を書いてください。

質問: {question}
""")

hypothetical_chain = hypothetical_prompt | model | StrOutputParser()

hyde_rag_chain = {
    "question": RunnablePassthrough(),
    "context": hypothetical_chain | retriever,
} | prompt | model | StrOutputParser()

result = hyde_rag_chain.invoke("LangChainの概要を教えて")

print(result)
```

出力

```txt
LangChainは、AIアプリケーションにおいて中心的なコンポーネントのための共通インターフェースを提供するフレームワークです。特に、チャットモデルに関しては、すべてのチャットモデルが「BaseChatModel」インターフェースを実装しており、これによりチャットモデルとの標準的なインタラクションが可能になります。LangChainは、ツール呼び出しや構造化出力など、プロバイダー特有の重要な機能をサポートしています。

また、LangChainは異なるプロバイダーのチャットモデルを一貫して扱うためのインターフェースを提供し、アプリケーションの監視、デバッグ、最適化のための追加機能も備えています。さらに、LangChainはチャットモデルやベクトルストアなどのコンポーネントを迅速に利用できるようにするための統合リストも用意しています。
```

最初の仮説の責任が重い気がするな～。実際にこのケースが使えるのは LLM が仮説的な内容を推論しやすいケースみたい

<details close markdown="1">
  <summary>プログラム全文</summary>

```py
from langchain_openai import ChatOpenAI
from langchain_core.runnables import RunnablePassthrough
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import OpenAIEmbeddings
from langchain_chroma import Chroma
from langchain_community.document_loaders import GitLoader
from langchain_text_splitters import CharacterTextSplitter
from dotenv import load_dotenv

load_dotenv()


def file_filter(file_path: str) -> bool:
    return file_path.endswith(".mdx")


loader = GitLoader(
    clone_url="https://github.com/langchain-ai/langchain",
    repo_path="./langchain",
    branch="master",
    file_filter=file_filter,
)

documents = loader.load()

text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

print(len(docs))

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
db = Chroma.from_documents(docs, embeddings)


prompt = ChatPromptTemplate.from_template('''\
以下の文脈だけを踏まえて質問に回答してください。

文脈: """
{context}
"""

質問: {question}
''')

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)

retriever = db.as_retriever()

hypothetical_prompt = ChatPromptTemplate.from_template("""\
次の質問に回答する一文を書いてください。

質問: {question}
""")

hypothetical_chain = hypothetical_prompt | model | StrOutputParser()

hyde_rag_chain = {
    "question": RunnablePassthrough(),
    "context": hypothetical_chain | retriever,
} | prompt | model | StrOutputParser()

result = hyde_rag_chain.invoke("LangChainの概要を教えて")

print(result)
```

</details>

#### 複数の検索クエリの生成

ユーザーの検索クエリから複数の検索クエリを生成してそれをもとに検索を行い最終的に一つにまとめる。

```py
from pydantic import BaseModel, Field


class QueryGenerationOutput(BaseModel):
    queries: list[str] = Field(..., description="検索クエリのリスト")


query_generation_prompt = ChatPromptTemplate.from_template("""\
質問に対してベクターデータベースから関連文書を検索するために、
3つの異なる検索クエリを生成してください。
距離ベースの類似性検索の限界を克服するために、
ユーザーの質問に対して複数の視点を提供することが目標です。

質問: {question}
""")

query_generation_chain = (
    query_generation_prompt
    | model.with_structured_output(QueryGenerationOutput)
    | (lambda x: x.queries)
)

multi_query_rag_chain = {
    "question": RunnablePassthrough(),
    "context": query_generation_chain | retriever.map(),
} | prompt | model | StrOutputParser()

multi_query_rag_chain.invoke("LangChainの概要を教えて")
```

<details close markdown="1">
  <summary>プログラム全文</summary>

```py
from pydantic import BaseModel, Field
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.runnables import RunnablePassthrough
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import OpenAIEmbeddings
from langchain_chroma import Chroma
from langchain_community.document_loaders import GitLoader
from langchain_text_splitters import CharacterTextSplitter
from dotenv import load_dotenv
load_dotenv()


def file_filter(file_path: str) -> bool:
    return file_path.endswith(".mdx")


loader = GitLoader(
    clone_url="https://github.com/langchain-ai/langchain",
    repo_path="./langchain",
    branch="master",
    file_filter=file_filter,
)

documents = loader.load()

text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

print(len(docs))

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
db = Chroma.from_documents(docs, embeddings)


class QueryGenerationOutput(BaseModel):
    queries: list[str] = Field(..., description="検索クエリのリスト")


prompt = ChatPromptTemplate.from_template('''\
以下の文脈だけを踏まえて質問に回答してください。

文脈: """
{context}
"""

質問: {question}
''')

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)
retriever = db.as_retriever()

query_generation_prompt = ChatPromptTemplate.from_template("""\
質問に対してベクターデータベースから関連文書を検索するために、
3つの異なる検索クエリを生成してください。
距離ベースの類似性検索の限界を克服するために、
ユーザーの質問に対して複数の視点を提供することが目標です。

質問: {question}
""")

query_generation_chain = (
    query_generation_prompt
    | model.with_structured_output(QueryGenerationOutput)
    | (lambda x: x.queries)
)
multi_query_rag_chain = {
    "question": RunnablePassthrough(),
    "context": query_generation_chain | retriever.map(),
} | prompt | model | StrOutputParser()

result = multi_query_rag_chain.invoke("LangChainの概要を教えて")

print(result)

```

</details>

出力

```txt
LangChainは、開発者が推論を行うアプリケーションを簡単に構築できるようにすることを目的としたPythonパッケージおよび企業です。元々は単一のオープンソースパッケージとして始まりましたが、現在は企業とエコシステム全体に進化しています。LangChainのエコシステム内の多くのコンポーネントは独立して使用できるため、特定のコンポーネントに特に魅力を感じる場合は、それを選んで使用することができます。

LangChainは、AIアプリケーションにおいて中心的な役割を果たすコンポーネントのための共通インターフェースを提供します。例えば、すべてのチャットモデルは「BaseChatModel」インターフェースを実装しており、これによりチャットモデルとの標準的なインタラクションが可能になります。LangChainは、ツール呼び出しや構造化出力などの重要な機能をサポートしています。

LangChainの主な特徴には、以下のようなものがあります：
- プロバイダーの交換が容易であること
- 高度な機能の提供（ストリーミングやツール呼び出しなど）

また、LangChainは複数のオープンソースライブラリで構成されており、各コンポーネントは「langchain-core」などの基本抽象クラスのサブクラスとして実装されています。これにより、開発者は自分のユースケースに最適なコンポーネントを選択して利用することができます。
```

LangSmith を見てみると複数の検索クエリが出されていることがわかる。

![image]({{site.baseurl}}/images/reading/rag_aiagent_nyuumon/06_multiquery.png)

### 6.4 検索後の工夫

#### RAG-Fusion

複数の検索結果を融合して並び替えるアルゴリズムとして RRF(Reciprocal Rank Fusion)というものがある。
各検索クエリに順位付けをして、「1/(順位+k)」(k はパラメータでよく 60 が使われるみたい)でスコアを計算して出力するみたい。よくわからん。

```py
from langchain_core.documents import Document


def reciprocal_rank_fusion(
    retriever_outputs: list[list[Document]],
    k: int = 60,
) -> list[str]:
    # 各ドキュメントのコンテンツ (文字列) とそのスコアの対応を保持する辞書を準備
    content_score_mapping = {}

    # 検索クエリごとにループ
    for docs in retriever_outputs:
        # 検索結果のドキュメントごとにループ
        for rank, doc in enumerate(docs):
            content = doc.page_content

            # 初めて登場したコンテンツの場合はスコアを0で初期化
            if content not in content_score_mapping:
                content_score_mapping[content] = 0

            # (1 / (順位 + k)) のスコアを加算
            content_score_mapping[content] += 1 / (rank + k)

    # スコアの大きい順にソート
    ranked = sorted(content_score_mapping.items(), key=lambda x: x[1], reverse=True)  # noqa
    return [content for content, _ in ranked]

rag_fusion_chain = {
    "question": RunnablePassthrough(),
    "context": query_generation_chain | retriever.map() | reciprocal_rank_fusion,
} | prompt | model | StrOutputParser()

rag_fusion_chain.invoke("LangChainの概要を教えて")
```

なるほど。各検索クエリで複数のドキュメントが取得していてそれは関連度が高いものが最初に入っているのね。
それをスコアつけて合計している。だから、複数同じドキュメントが引っ掛かるとスコアも大きくなっていく。

<details close markdown="1">
  <summary>プログラム全文</summary>

```py
from langchain_core.documents import Document
from pydantic import BaseModel, Field
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.runnables import RunnablePassthrough
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import OpenAIEmbeddings
from langchain_chroma import Chroma
from langchain_community.document_loaders import GitLoader
from langchain_text_splitters import CharacterTextSplitter
from dotenv import load_dotenv
load_dotenv()


def file_filter(file_path: str) -> bool:
    return file_path.endswith(".mdx")


loader = GitLoader(
    clone_url="https://github.com/langchain-ai/langchain",
    repo_path="./langchain",
    branch="master",
    file_filter=file_filter,
)

documents = loader.load()

text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
db = Chroma.from_documents(docs, embeddings)


class QueryGenerationOutput(BaseModel):
    queries: list[str] = Field(..., description="検索クエリのリスト")


prompt = ChatPromptTemplate.from_template('''\
以下の文脈だけを踏まえて質問に回答してください。

文脈: """
{context}
"""

質問: {question}
''')

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)
retriever = db.as_retriever()


def reciprocal_rank_fusion(
    retriever_outputs: list[list[Document]],
    k: int = 60,
) -> list[str]:
    # 各ドキュメントのコンテンツ (文字列) とそのスコアの対応を保持する辞書を準備
    content_score_mapping = {}

    # 検索クエリごとにループ
    for docs in retriever_outputs:
        # 検索結果のドキュメントごとにループ
        for rank, doc in enumerate(docs):
            content = doc.page_content

            # 初めて登場したコンテンツの場合はスコアを0で初期化
            if content not in content_score_mapping:
                content_score_mapping[content] = 0

            # (1 / (順位 + k)) のスコアを加算
            content_score_mapping[content] += 1 / (rank + k)

    # スコアの大きい順にソート
    ranked = sorted(content_score_mapping.items(), key=lambda x: x[1], reverse=True)  # noqa
    return [content for content, _ in ranked]


query_generation_prompt = ChatPromptTemplate.from_template("""\
質問に対してベクターデータベースから関連文書を検索するために、
3つの異なる検索クエリを生成してください。
距離ベースの類似性検索の限界を克服するために、
ユーザーの質問に対して複数の視点を提供することが目標です。

質問: {question}
""")

query_generation_chain = (
    query_generation_prompt
    | model.with_structured_output(QueryGenerationOutput)
    | (lambda x: x.queries)
)

rag_fusion_chain = {
    "question": RunnablePassthrough(),
    "context": query_generation_chain | retriever.map() | reciprocal_rank_fusion,
} | prompt | model | StrOutputParser()

result = rag_fusion_chain.invoke("LangChainの概要を教えて")

print(result)

```

</details>

出力

```txt
LangChainは、AIアプリケーションの中心となるコンポーネントのための共通インターフェースを提供するフレームワークです。開発者が推論を行うアプリケーションを簡単に構築できるようにすることを目指しています。元々はオープンソースの単一パッケージとして始まりましたが、現在は企業とエコシステム全体に進化しています。

LangChainの主な特徴には、以下が含まれます：

- **標準API**: すべてのチェーンはRunnableインターフェースを使用して構築されており、他のRunnableと同様に使用できます。
- **最適化された並列実行**: Runnablesを並列で実行することができ、処理のレイテンシを大幅に削減します。
- **非同期サポート**: LCELで構築されたチェーンは非同期に実行でき、大量のリクエストを同時に処理するのに役立ちます。
- **ストリーミングの簡素化**: チェーンの実行中に出力をストリーミングでき、最初のトークンが出力されるまでの時間を最小限に抑えることができます。

LangChainは、さまざまなオープンソースライブラリで構成されており、特定のコンポーネントを選んで使用することが可能です。これにより、開発者は自分のユースケースに最適なコンポーネントを自由に選択できます。
```

RAG-Fusion を使った出力の方がなんか読みやすい気はする。

#### リランクモデルの概要

RRF では複数の検索を行い得た結果をまとめた。この際に、特定の検索で得た結果を改めて並び替える（リランク）が有効な場合がある。

ここでは Cohere というものを使用してリランクをしてみる。

```py
from typing import Any

from langchain_cohere import CohereRerank
from langchain_core.documents import Document


def rerank(inp: dict[str, Any], top_n: int = 3) -> list[Document]:
    question = inp["question"]
    documents = inp["documents"]

    cohere_reranker = CohereRerank(model="rerank-multilingual-v3.0", top_n=top_n)
    return cohere_reranker.compress_documents(documents=documents, query=question)


rerank_rag_chain = (
    {
        "question": RunnablePassthrough(),
        "documents": retriever,
    }
    | RunnablePassthrough.assign(context=rerank)
    | prompt | model | StrOutputParser()
)

rerank_rag_chain.invoke("LangChainの概要を教えて")
```

出力

```txt
LangChainは、AIアプリケーションにおいて中心的なコンポーネントのための共通インターフェースを提供するフレームワークです。これにより、異なるプロバイダー特有の機能（例えば、ツール呼び出しや構造化出力）をサポートしつつ、チャットモデルなどのコンポーネントと標準的に対話することが可能になります。LangChainのエコシステムは、パッケージが整理されており、さまざまなAIアプリケーションの構築を容易にします。具体的な実装例や手順は、チュートリアルやハウツーガイドで提供されています。
```

こんな感じで順番が少し変わっている

![image]({{site.baseurl}}/images/reading/rag_aiagent_nyuumon/06_rerank.png)

### 6.5 複数の Retriever を使う工夫

いままで使用していた Retriever に加えて、Web 用の Retriever の用意

```py
from langchain_community.retrievers import TavilySearchAPIRetriever

langchain_document_retriever = retriever.with_config(
    {"run_name": "langchain_document_retriever"}
)

web_retriever = TavilySearchAPIRetriever(k=3).with_config(
    {"run_name": "web_retriever"}
)

```

Retriever を選択するプロンプトを設定

```py
from enum import Enum


class Route(str, Enum):
    langchain_document = "langchain_document"
    web = "web"


class RouteOutput(BaseModel):
    route: Route


route_prompt = ChatPromptTemplate.from_template("""\
質問に回答するために適切なRetrieverを選択してください。

質問: {question}
""")

route_chain = (
    route_prompt
    | model.with_structured_output(RouteOutput)
    | (lambda x: x.route)
)
```

選択した結果を踏まえて結果を出す

```py
def routed_retriever(inp: dict[str, Any]) -> list[Document]:
    question = inp["question"]
    route = inp["route"]

    if route == Route.langchain_document:
        return langchain_document_retriever.invoke(question)
    elif route == Route.web:
        return web_retriever.invoke(question)

    raise ValueError(f"Unknown route: {route}")


route_rag_chain = (
    {
        "question": RunnablePassthrough(),
        "route": route_chain,
    }
    | RunnablePassthrough.assign(context=routed_retriever)
    | prompt | model | StrOutputParser()
)

route_rag_chain.invoke("LangChainの概要を教えて")
```

<details close markdown="1">
  <summary>プログラム全文</summary>

```py
from enum import Enum
from langchain_community.retrievers import TavilySearchAPIRetriever
from langchain_cohere import CohereRerank
from typing import Any
from langchain_core.documents import Document
from pydantic import BaseModel, Field
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.runnables import RunnablePassthrough
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import OpenAIEmbeddings
from langchain_chroma import Chroma
from langchain_community.document_loaders import GitLoader
from langchain_text_splitters import CharacterTextSplitter
from dotenv import load_dotenv
load_dotenv()


def file_filter(file_path: str) -> bool:
    return file_path.endswith(".mdx")


loader = GitLoader(
    clone_url="https://github.com/langchain-ai/langchain",
    repo_path="./langchain",
    branch="master",
    file_filter=file_filter,
)

documents = loader.load()

text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
db = Chroma.from_documents(docs, embeddings)


prompt = ChatPromptTemplate.from_template('''\
以下の文脈だけを踏まえて質問に回答してください。

文脈: """
{context}
"""

質問: {question}
''')

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)
retriever = db.as_retriever()


langchain_document_retriever = retriever.with_config(
    {"run_name": "langchain_document_retriever"}
)

web_retriever = TavilySearchAPIRetriever(k=3).with_config(
    {"run_name": "web_retriever"}
)


class Route(str, Enum):
    langchain_document = "langchain_document"
    web = "web"


class RouteOutput(BaseModel):
    route: Route


route_prompt = ChatPromptTemplate.from_template("""\
質問に回答するために適切なRetrieverを選択してください。

質問: {question}
""")

route_chain = (
    route_prompt
    | model.with_structured_output(RouteOutput)
    | (lambda x: x.route)
)


def routed_retriever(inp: dict[str, Any]) -> list[Document]:
    question = inp["question"]
    route = inp["route"]

    if route == Route.langchain_document:
        return langchain_document_retriever.invoke(question)
    elif route == Route.web:
        return web_retriever.invoke(question)

    raise ValueError(f"Unknown route: {route}")


route_rag_chain = (
    {
        "question": RunnablePassthrough(),
        "route": route_chain,
    }
    | RunnablePassthrough.assign(context=routed_retriever)
    | prompt | model | StrOutputParser()
)

question = "LangChainの概要を教えて"
result = route_rag_chain.invoke(question)
print(result)

print("-----------------------------------------")

question = "東京の今日の天気は？"
result = route_rag_chain.invoke(question)
print(result)

```

</details>

出力

```txt
LangChainは、AIアプリケーションにおいて中心的なコンポーネントのための共通インターフェースを提供するフレームワークです。これにより、異なるプロバイダー特有の機能（例えば、ツール呼び出しや構造化出力）をサポートしつつ、チャットモデルなどのコンポーネントと標準的に対話することが可能になります。また、LangChainはカスタムチェーンを作成するための「LangChain Expression Language」（LCEL）を提供しており、これによりユーザーは柔軟なチェーンを構築できます。さらに、LangChainの概念ガイドは、フレームワークの背後にある重要な概念を説明し、ユーザーがより深い理解を得る手助けをします。
-----------------------------------------
東京の今日の天気は曇り時々雨で、雷を伴う所がある見込みです。湿った空気の影響を受けており、特に夜のはじめ頃まで雷を伴う可能性があります。
```

LangSmith を確認してみると Retriever を選択している様子がわかる

![image]({{site.baseurl}}/images/reading/rag_aiagent_nyuumon/06_multiretriever.png)

#### ハイブリット検索の例

先ほどはどちらかの Retriever を使用しての検索だった。しかし、複数の Retriever を組み合わせたい場合もある。例えば、ベクトル化する際に、
汎用的に作成された Embedding モデルよりも単語の登場頻度をベースに類似度を測定する TF-IDF や BM25(Elasiticsearch のデフォルトアルゴリズム)の方が有効な場合がある。
特徴として、一般的な Embedding モデルは密なベクトルになるのに対し、TF-IDF や BM25 は疎なベクトルになる。
このような二つのアルゴリズムがから作成された Retriever を組み合わせて検索を行う。

## 第 7 章 LangSmith を使った RAG アプリケーションの評価

### 7.1 第 7 章で取り組む評価の概要

- オフライン評価：あらかじめ用意したデータセットを使用した評価
- オンライン評価：実ユーザーの反応など、実際のトラフィックを使った評価

### 7.3 LangSmith と Ragas を使ったオフライン評価の構成例

Ragas: RAG の評価フレームワークで Github で OSS として公開されている。[github のリンク](https://github.com/explodinggradients/ragas)

### 7.4 Ragas による合成テストデータの生成

ragas でテストデータを作成して langsmith にデータセットとして格納する保存する。
このようにデータが格納される。

![image]({{site.baseurl}}/images/reading/rag_aiagent_nyuumon/07_ragas.png)

中身を見てみる。

一つ目の例

- Question: What are the constraints for ToolMessages in few-shot prompting with
  AIMessages and HumanMessages?
- Ground Truth: The context does not provide specific constraints for ToolMessages
  in few-shot prompting with AIMessages and HumanMessages.

ToolMessages の制約を聞いて、なんも制約ないという回答。。。なんかいまいち

二つ目の例

- Question: What types of CRM and analytics apps does Salesforce provide?
- Ground Truth: Salesforce provides customer relationship management (CRM)
  solutions and a suite of enterprise applications focused on sales, customer
  service, marketing automation, and analytics.

Salseforce はどんなものを提供しているの？という問いに営業支援やカスタマーサービスを提供しているという回答。

他もそれっぽい質問と回答ができていた。ただ、十分な品質かどうかは限らないため、Ragas のプロンプトを参考にしつつ自前で
データを生成することもがんが得られるらしい。

<details close markdown="1">
  <summary>プログラム全文</summary>

```py
from langsmith import Client
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from ragas.testset.evolutions import simple, reasoning, multi_context
from ragas.testset.generator import TestsetGenerator
import nest_asyncio
from langchain_community.document_loaders import GitLoader
from dotenv import load_dotenv
load_dotenv()


def file_filter(file_path: str) -> bool:
    return file_path.endswith(".mdx")


loader = GitLoader(
    clone_url="https://github.com/langchain-ai/langchain",
    repo_path="./langchain",
    branch="master",
    file_filter=file_filter,
)

documents = loader.load()
print(len(documents))

for document in documents:
    document.metadata["filename"] = document.metadata["source"]


nest_asyncio.apply()

generator = TestsetGenerator.from_langchain(
    generator_llm=ChatOpenAI(model="gpt-4o-mini"),
    critic_llm=ChatOpenAI(model="gpt-4o-mini"),
    embeddings=OpenAIEmbeddings(),
)

testset = generator.generate_with_langchain_docs(
    documents,
    test_size=4,
    distributions={simple: 0.5, reasoning: 0.25, multi_context: 0.25},
)

result = testset.to_pandas()

print(result)

inputs = []
outputs = []
metadatas = []

for testset_record in testset.test_data:
    inputs.append(
        {
            "question": testset_record.question,
        }
    )
    outputs.append(
        {
            "contexts": testset_record.contexts,
            "ground_truth": testset_record.ground_truth,
        }
    )
    metadatas.append(
        {
            "source": testset_record.metadata[0]["source"],
            "evolution_type": testset_record.evolution_type,
        }
    )

dataset_name = "agent-book"

client = Client()

if client.has_dataset(dataset_name=dataset_name):
    client.delete_dataset(dataset_name=dataset_name)

dataset = client.create_dataset(dataset_name=dataset_name)

client.create_examples(
    inputs=inputs,
    outputs=outputs,
    metadata=metadatas,
    dataset_id=dataset.id,
)
```

</details>

### 7.5 LangSmith と Ragas を使ったオフライン評価の実装

評価をする際は下記の 3 つのものを指定する

- 推論の関数
- Dataset の名前
- Evaluator（評価機）

Evaluator（評価機）は LangChain が提供している機能を使用できる。また、独自に定義することも可能。
今回は Ragas の評価メトリクスを使用したものを独自で作成する。

Ragas の評価メトリクスの観点

- 「検索」の評価メトリクス
  - Context recall: 質問と期待する回答より、有用な結果だと LLM で判断される割合
  - Context precision: 期待する回答をいくつかの文章に分割し、実際の検索結果で説明できる割合
  - Context entity recall: 期待する回答に含まれるエンティティのうち、実際の検索結果に含まれる割合
- 「生成」の評価メトリクス
  - Faithfulness: 実際の解答が質問にどれだけ関連するか
  - Answer relevancy: 実際の解答に含まれる主張のうち、実際にの検索結果と一貫している割合
- 「検索 + 生成」の評価メトリクス
  - Ansewr similarity: 実際の解答と期待する回答の埋め込みベクトルのコサイン類似度
  - Answer correctness: 実際の解答と期待する回答の事実的類似性 (= F1 スコア) と意味的類似姿勢の加重平均

これらは LLM や Embedding を使って実装されている。

#### カスタム Evaluator の実装

カスタム Evaluator は実際の実行結果(`Run`)と評価データ(`Example`)を引数として評価スコアを`dict`で返す関数として実装できる。
Ragas の評価メトリクスを使用する場合は LLM や Embedding モデルを設定する必要がある。

下記のコードは Ragas の評価メトリクスを LangSmith での評価に使うためのカスタム Evaluator の実装例（[参考](https://docs.ragas.io/en/stable/howtos/integrations/_langfuse/#the-metrics)）。

重要箇所は`score`のメソッドをして評価スコアを算出している箇所で、実際の実行結果(`Run`)と評価データ(`Example`)から取り出した下記内容を渡している

- `question`：質問
- `answer`：実際の実行結果
- `context`：実際の検索結果
- `ground_truth`：期待する回答

```py
from typing import Any

from langchain_core.embeddings import Embeddings
from langchain_core.language_models import BaseChatModel
from langsmith.schemas import Example, Run
from ragas.embeddings import LangchainEmbeddingsWrapper
from ragas.llms import LangchainLLMWrapper
from ragas.metrics.base import Metric, MetricWithEmbeddings, MetricWithLLM


class RagasMetricEvaluator:
    def __init__(self, metric: Metric, llm: BaseChatModel, embeddings: Embeddings):
        self.metric = metric

        # LLMとEmbeddingsをMetricに設定
        if isinstance(self.metric, MetricWithLLM):
            self.metric.llm = LangchainLLMWrapper(llm)
        if isinstance(self.metric, MetricWithEmbeddings):
            self.metric.embeddings = LangchainEmbeddingsWrapper(embeddings)

    def evaluate(self, run: Run, example: Example) -> dict[str, Any]:
        context_strs = [doc.page_content for doc in run.outputs["contexts"]]

        # Ragasの評価メトリクスのscoreメソッドでスコアを算出
        score = self.metric.score(
            {
                "question": example.inputs["question"],
                "answer": run.outputs["answer"],
                "contexts": context_strs,
                "ground_truth": example.outputs["ground_truth"],
            },
        )
        return {"key": self.metric.name, "score": score}
```

このコードは下記コードのように呼び出す

```py
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from ragas.metrics import answer_relevancy, context_precision

metrics = [context_precision, answer_relevancy]

llm = ChatOpenAI(model="gpt-4o", temperature=0)
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

evaluators = [
    RagasMetricEvaluator(metric, llm, embeddings).evaluate
    for metric in metrics
]
```

#### 推論の関数の実装

推論の関数の実装例

シンプルな RAG の chain を実装して、その実際の検索結果や回答を返す関数`predict`を実装する

```py
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableParallel, RunnablePassthrough
from langchain_openai import ChatOpenAI

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
db = Chroma.from_documents(documents, embeddings)

prompt = ChatPromptTemplate.from_template('''\
以下の文脈だけを踏まえて質問に回答してください。

文脈: """
{context}
"""

質問: {question}
''')

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)

retriever = db.as_retriever()

chain = RunnableParallel(
    {
        "question": RunnablePassthrough(),
        "context": retriever,
    }
).assign(answer=prompt | model | StrOutputParser())

# 推論の関数
def predict(inputs: dict[str, Any]) -> dict[str, Any]:
    question = inputs["question"]
    output = chain.invoke(question)
    return {
        "contexts": output["context"],
        "answer": output["answer"],
    }
```

#### オフライン評価の実装・実行

LangSmith でオフライン評価を実行するコード

```py
from langsmith.evaluation import evaluate

evaluate(
    predict,
    data="agent-book",
    evaluators=evaluators,
)
```

<details close markdown="1">
  <summary>プログラム全文</summary>

```py
from langchain_text_splitters import CharacterTextSplitter
from langsmith.evaluation import evaluate
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_openai import ChatOpenAI
from langchain_core.runnables import RunnableParallel, RunnablePassthrough
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import OpenAIEmbeddings
from langchain_community.document_loaders import GitLoader
from langchain_chroma import Chroma
from ragas.metrics import answer_relevancy, context_precision
from typing import Any

from langchain_core.embeddings import Embeddings
from langchain_core.language_models import BaseChatModel
from langsmith.schemas import Example, Run
from ragas.embeddings import LangchainEmbeddingsWrapper
from ragas.llms import LangchainLLMWrapper
from ragas.metrics.base import Metric, MetricWithEmbeddings, MetricWithLLM
from dotenv import load_dotenv
import asyncio
import nest_asyncio
nest_asyncio.apply()
load_dotenv()


class RagasMetricEvaluator:
    def __init__(self, metric: Metric, llm: BaseChatModel, embeddings: Embeddings):
        self.metric = metric

        # LLMとEmbeddingsをMetricに設定
        if isinstance(self.metric, MetricWithLLM):
            self.metric.llm = LangchainLLMWrapper(llm)
        if isinstance(self.metric, MetricWithEmbeddings):
            self.metric.embeddings = LangchainEmbeddingsWrapper(embeddings)

    def evaluate(self, run: Run, example: Example) -> dict[str, Any]:
        context_strs = [doc.page_content for doc in run.outputs["contexts"]]

        # Ragasの評価メトリクスのscoreメソッドでスコアを算出
        score = self.metric.score(
            {
                "question": example.inputs["question"],
                "answer": run.outputs["answer"],
                "contexts": context_strs,
                "ground_truth": example.outputs["ground_truth"],
            },
        )
        return {"key": self.metric.name, "score": score}


metrics = [context_precision, answer_relevancy]

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

evaluators = [
    RagasMetricEvaluator(metric, llm, embeddings).evaluate
    for metric in metrics
]


def file_filter(file_path: str) -> bool:
    return file_path.endswith(".mdx")


loader = GitLoader(
    clone_url="https://github.com/langchain-ai/langchain",
    repo_path="./langchain",
    # branch="master",
    branch="langchain==0.2.13",
    file_filter=file_filter,
)

documents = loader.load()

text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
db = Chroma.from_documents(docs, embeddings)

prompt = ChatPromptTemplate.from_template('''\
以下の文脈だけを踏まえて質問に回答してください。

文脈: """
{context}
"""

質問: {question}
''')

model = ChatOpenAI(model="gpt-4o-mini", temperature=0)

retriever = db.as_retriever()

chain = RunnableParallel(
    {
        "question": RunnablePassthrough(),
        "context": retriever,
    }
).assign(answer=prompt | model | StrOutputParser())


def predict(inputs: dict[str, Any]) -> dict[str, Any]:
    question = inputs["question"]
    output = chain.invoke(question)
    return {
        "contexts": output["context"],
        "answer": output["answer"],
    }


evaluate(
    predict,
    data="agent-book",
    evaluators=evaluators,
)
```

</details>

実際にの LangSmith の画面がこんな感じ

![image]({{site.baseurl}}/images/reading/rag_aiagent_nyuumon/07_ragas_langsmith01.png)

各項目の説明はこんな感じ

| 項目名                | 種別           | 意味                                                                                            | 値の範囲 | 理想的な値                                     | 備考                                                                     |
| --------------------- | -------------- | ----------------------------------------------------------------------------------------------- | -------- | ---------------------------------------------- | ------------------------------------------------------------------------ |
| **Splits**            | データ分割     | 実験で使用したデータセットの分割方法（例：`train`, `test`, `base`など）                         | -        | -                                              | 評価対象がどのデータ分割であるかを表すだけ。モデル評価に直接影響はない。 |
| **Num Repetitions**   | 実験回数       | 同じ設定で実験を何回繰り返したか（平均値を取ることもある）                                      | 整数値   | 1 以上                                         | 値が大きいほど安定した平均値になる。                                     |
| **answer_relevancy**  | 回答品質       | 生成された**回答が質問にどれだけ関連しているか**（＝回答がちゃんと質問に答えているか）          | 0〜1     | **1 に近いほど良い**                           | 0 なら全く関係ない回答、1 なら完全に関連している回答。                   |
| **context_precision** | 検索精度       | 取得した**コンテキスト（文書）に、実際に回答に必要な情報がどれだけ含まれているか**              | 0〜1     | **1 に近いほど良い**                           | Retriever が関係のない文書を混ぜていると下がる。Retriever 改善で上がる。 |
| **P50 Latency**       | 性能（速度）   | **レスポンス時間の中央値（50 パーセンタイル）**。全回答のうち半分がこの時間以下で応答している。 | 秒（s）  | **小さいほど良い**                             | 実際の RAG の応答速度。3 秒程度なら実用範囲内。                          |
| **P99 Latency**       | 性能（安定性） | **最も遅い 1%の応答時間**。つまり、ほぼ最悪ケースの遅さ。                                       | 秒（s）  | **P50 より大きいが、あまり大きくない方が良い** | 高負荷時や長文入力で遅くなるケースを示す。                               |
| **Error Rate**        | 成功率         | 実験の失敗（API エラーやタイムアウトなど）が発生した割合                                        | 0〜100%  | **0%が理想**                                   | API エラー、ネットワーク切断などで上昇。                                 |

answer_relevancy と context_precision の解説

- answer_relevancy：最終的に生成された回答に対する評価で、質問内容に対してどれだけ適した回答か評価
  - 改善方法
    - context_precision の改善
    - LLM および、Prompt 設計の改善
- context_precision：Retriever が取得したコンテキスト（文書群）に対する評価で、質問に回答するのに必要な情報がどの程度含まれているのか評価
  - 改善方法
    - Retriever の改善
    - Embedding 設定の変更

つまり、context_precision が低いと良い回答を生成できたないため、answer_relevancy も悪くなる。
一方で、context_precision が良いのに answer_relevancy が悪い場合は、生成モデルがうまく回答を生成できていないことが考えられる。

一般的に、answer_relevancy, context_precision は 0.8 ~ 1.0 が優秀らしい、0.5 を下回ると改善が必要。
つまり今回の場合だと context_precision が 0.5（ギリギリで普通）で answer_relevancy が 0.22 なので回答生成の精度が低く改善が必要となる。

オフライン評価をする場合は、本番データとはギャップがあることを認識しておく必要がある。
オフライン評価がうまくいっても本番データではうまくいかないことはある。そこを踏まえて、本番環境での実際のユーザーの反応を確認することは重要。

### 7.6 LangSmith を使ったフィードバックの収集

Google Colab 用のコードだったので一旦パス。結構簡単に langsmith に good,bad の情報を格納できる。
また、good のデータを dataset として自動で登録することもでる

## 第 8 章 AI エージェントのための LLM 活用の期待

### 8.1 AI エージェントのための LLM 活用の期待

AI エージェント

- 複雑なタスクを自立的に遂行できる AI システム
- 人の指示なし、もしくは最低限の指示で AI 自身がすることを判断することが求められている

### 8.2 AI エージェントの期限と LLM を使った AI エージェントの変遷

エージェントアプローチ人工知能（1995 年）の中ですでに「エージェントとは、環境を認識し、目標を達成するために自律的に行動する存在」と定義されている。

- 1990 年代: 統計的言語モデル（過去の文脈から次の単語の予想）
  - n-gram 言語モデル
  - 統計的手法
  - 確率推論
- 2013 年: ニューラル言語モデル（単語の周辺の文脈より意味を予想）
  - Word2Vec (NPLM)
  - NLPS
- 2018 年: 事前学習済み言語モデル（自然言語での問題解決能力の向上）
  - ELMO
  - BERT
  - GPT-1/2
  - 事前学習＋ファインチューニング
- 2020 年: 大規模言語モデル (LLM)（自然言語に限らない様々な問題解決能力の向上）
  - GPT-3/4
  - ChatGPT
  - Claude

#### WebGPT

- 2021 年に OpenAI の研究者より発表
- GPT-3 をファインチューニングした LLM
- 56%の解答が人間のデモンストレーターと比較して好ましいと評価
- ファインチューニングにより信頼性のばらつきがある情報ソースでも有益な回答を生成できる有用性が示された

#### Chain-of-Thought プロンプティング

2022 年に複雑な推論に対して中間的な推論ステップの実例を示すことで推論能力の向上が見込めることが示された。

Chain-of-Thought プロンプティングの例

```text
Q: Aさんはテニスボールを5個持っています。彼はあと2缶のテニスボールを買いました。それぞれの缶には3つづつテニスボールが入っています。現在彼はいくつのテニスボールを持ってますでしょうか？

A: 初めにAさんは5つのボールを持っています。2つの缶に3つずつボールがあるのは合わせて6つのてしぼーるということです。5 + 6 = 11です。

Q: カフェテリアにリンゴが23個あります。20個をランチのために使って、6個を買い足したとき、いくつのリンゴがあるでしょうか？
```

このようなプロンプトは Chain-of-Thought プロンプティング、または Few-Shot Chain-of-Thought プロンプティングと呼ばれる。
また、Zero-shot CoT プロンプティングというものもあり、それは「ステップバイステップで考えましょう」とつけるものになる。

Zero-shot CoT プロンプティングの例

```text
Q: Aさんはテニスボールを5個持っています。彼はあと2缶のテニスボールを買いました。それぞれの缶には3つづつテニスボールが入っています。現在彼はいくつのテニスボールを持ってますでしょうか？

A: ステップバイステップで考えましょう。
```

#### MRKL Systems

2022 年に発表された MRKL（ミラクル）Systems という論文および実装では小規模で特化型の言語モデルやデータベースなどの外部接続 API をモジュールとして構成し、
LLM により最適なモジュールへルーティングすることで専門性の高い出力を処理できることが示された。

#### Reasoning and Acting (ReAct)

行動計画の作成や調整を行う Reasoning（推論）工程と外部から必要な情報を取り込んだ入り、目的の外部実行を行う Acting（行動）工程、そしてこの二つを交互に繰り返し、
両者の相乗効果を引き出す手法

#### Plan-and-Solve プロンプティング

Zero-shot CoT でも計算ミスや中間推論ステップの欠如などがあり、有益な回答が得られない場合がある。
そこで、2023 年にあらかじめ計画を立ててから（タスクをサブタスクにぶんかいする）、計画に従ってサブタスクを実行する Plan-and-Solve プロンプティングという手法が提案された。

Plan-and-Solve プロンプティングの例

```text
Q: 生徒が20人いるダンスクラスで20%はコンテンポラリーダンスに登録し、それ以外はヒップホップに登録しました。ヒップホップに登録した生徒は何%でしょうか？

A: 恥に見問題を理解し、問題を解決する計画に分割しましょう。その後、各プランを実行して問題をステップバイステップで解決しましょう。
```

### 8.3 汎用 LLM エージェントのフレームワーク
