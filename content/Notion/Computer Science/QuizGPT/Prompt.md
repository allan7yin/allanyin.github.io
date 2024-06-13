```Plain
Generate a multiple choice quiz of 5 questions, where there are 4 options for each question, of the difficulty "medium" from the following prompt: Addition.

Also generate the correct answers for each of the questions generated from the prompt.
Also, generate each option on a new line. After all the options have been outputed, output the correction option on a new line.

Let the formatting be "Question: ..." then a new line, and then with the possible options beginning with the letter A: "A: ..." then a new line, the "B: ...", etc. At the end, have the answer as "Answer: ..." for each of the questions. After each group of question, options, and answer, print 2 new lines to separate from the next group.
```

With the above prompt, we get the following JSON:

```JSON
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "logprobs": null,
      "text": "\\nQuestion 1: What is the result of 75 + 16?\\nA: 91\\nB: 81\\nC: 89\\nD: 71\\nAnswer: A\\n\\nQuestion 2: What is the result of 67 + 97?\\nA: 154\\nB: 164\\nC: 174\\nD: 144\\nAnswer: B\\n\\nQuestion 3: What is the result of 57 + 37?\\nA: 94\\nB: 76\\nC: 84\\nD: 92\\nAnswer: A\\n\\nQuestion 4: What is the result of 16 + 17?\\nA: 33\\nB: 31\\nC: 37\\nD: 34\\nAnswer: A\\n\\nQuestion 5: What is the result of 73 + 97?\\nA: 170\\nB: 160\\nC: 150\\nD: 180\\nAnswer: A"
    }
  ],
  "created": 1686432460,
  "id": "cmpl-7Q0PUWmJZqoKTVr2Q4M5VSXkFBNKh",
  "model": "text-davinci-003",
  "object": "text_completion",
  "usage": {
    "completion_tokens": 169,
    "prompt_tokens": 166,
    "total_tokens": 335
  }
}
```

Parse this to obtain formatted text which will be returned into the response queue. First, we know questions are separated by the 2 new lines. We will filter through based on that condition:

```Plain
"\\nQuestion 1: What is the result of 75 + 16?\\nA: 91\\nB: 81\\nC: 89\\nD: 71\\nAnswer: A

Question 2: What is the result of 67 + 97?\\nA: 154\\nB: 164\\nC: 174\\nD: 144\\nAnswer: B

Question 3: What is the result of 57 + 37?\\nA: 94\\nB: 76\\nC: 84\\nD: 92\\nAnswer: A

Question 4: What is the result of 16 + 17?\\nA: 33\\nB: 31\\nC: 37\\nD: 34\\nAnswer: A

Question 5: What is the result of 73 + 97?\\nA: 170\\nB: 160\\nC: 150\\nD: 180\\nAnswer: A"
```

```Plain
Quiz(id=aa8ac98d-1cfb-4bc9-b373-3a2a18e1509a, Questions=[Question(questionId=97, text=Question 1: What is the result of 75 + 16?, options=[Option(optionId=231, text=A: 91, value=null, ordering=0), Option(optionId=232, text=B: 81, value=null, ordering=0), Option(optionId=233, text=C: 89, value=null, ordering=0), Option(optionId=234, text=D: 71, value=null, ordering=0)], answers=[Answer(answerId=66, text=Answer: A, value=null)]), Question(questionId=98, text=Question 2: What is the result of 67 + 97?, options=[Option(optionId=235, text=A: 154, value=null, ordering=0), Option(optionId=236, text=B: 164, value=null, ordering=0), Option(optionId=237, text=C: 174, value=null, ordering=0), Option(optionId=238, text=D: 144, value=null, ordering=0)], answers=[Answer(answerId=67, text=Answer: B, value=null)]), Question(questionId=99, text=Question 3: What is the result of 57 + 37?, options=[Option(optionId=239, text=A: 94, value=null, ordering=0), Option(optionId=240, text=B: 76, value=null, ordering=0), Option(optionId=241, text=C: 84, value=null, ordering=0), Option(optionId=242, text=D: 92, value=null, ordering=0)], answers=[Answer(answerId=68, text=Answer: A, value=null)]), Question(questionId=100, text=Question 4: What is the result of 16 + 17?, options=[Option(optionId=243, text=A: 33, value=null, ordering=0), Option(optionId=244, text=B: 31, value=null, ordering=0), Option(optionId=245, text=C: 37, value=null, ordering=0), Option(optionId=246, text=D: 34, value=null, ordering=0)], answers=[Answer(answerId=69, text=Answer: A, value=null)]), Question(questionId=101, text=Question 5: What is the result of 73 + 97?, options=[Option(optionId=247, text=A: 170, value=null, ordering=0), Option(optionId=248, text=B: 160, value=null, ordering=0), Option(optionId=249, text=C: 150, value=null, ordering=0), Option(optionId=250, text=D: 180, value=null, ordering=0)], answers=[Answer(answerId=70, text=Answer: A, value=null)])])
```