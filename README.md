![alt text](https://huggingface.co/blog/assets/86_bloom_megatron_deepspeed/bloom-banner.png)
## 🌲🤏 BLOOM-LoRA: Low-Rank LLaMA Instruct-Tuning

**Why do we try to finetune these BLOOM models? The major reason is the licence of LLaMA, while the BLOOM licence seems to be more relax!

**We try to reimplemt BLOOM-LoRA using a variety of sources such as [the original LLaMA](https://github.com/facebookresearch/llama), [Stanford-Alpaca](https://github.com/tatsu-lab/stanford_alpaca), [Alpaca-LoRA](https://github.com/tloen/alpaca-lora), [BLOOMZ](https://github.com/NouamaneTazi/bloomz.cpp), and a name to few.

**Try the pretrained model out on Colab [here](https://colab.research.google.com/drive/1LY5Ds6qyr_Drpp9WSdt-ZEMvvrFICdEx#scrollTo=VucO3HSMoJkz)!**

_**Update 2023-03-21:** weights have been updated with cleaned data and prompts masked out in the loss. This should reduce the number of template artifacts in outputs._

This repository contains code for reproducing the [Stanford Alpaca](https://github.com/tatsu-lab/stanford_alpaca) results using [low-rank adaptation (LoRA)](https://arxiv.org/pdf/2106.09685.pdf).
We provide an Instruct model of similar quality to `text-davinci-003` that can run [on a Raspberry Pi](https://twitter.com/miolini/status/1634982361757790209) (for research),
and the code can be easily extended to the `1b1`, `3b`, 7b1, and `175b` models.

In addition to the training code, which runs within five hours on a single RTX 4090, and A100
we publish a script for downloading and inference on the foundation model and LoRA,
as well as the resulting [LoRA weights themselves](https://huggingface.co/linhduongtuan/bloom-lora-560m/tree/main).
To fine-tune cheaply and efficiently, we use Hugging Face's [PEFT](https://github.com/huggingface/peft)
as well as Tim Dettmers' [bitsandbytes](https://github.com/TimDettmers/bitsandbytes).

Without hyperparameter tuning or validation-based checkpointing, the LoRA model produces outputs comparable to the Stanford Alpaca model. (Please see the outputs included below.) Further tuning might be able to achieve better performance; I invite interested users to give it a try and report their results.

For discussion and support, users have created a dedicated Twitter server [here](https://twitter.com/DuongTuanLinh1).

### Setup

1. Install dependencies

```
pip install -r requirements.txt
```

2. If bitsandbytes doesn't work, [install it from source.](https://github.com/TimDettmers/bitsandbytes/blob/main/compile_from_source.md) Windows users can follow [these instructions](https://github.com/tloen/alpaca-lora/issues/17).

### Inference (`generate.py`)

This file reads the foundation model from the Hugging Face model hub and the LoRA weights from `tloen/alpaca-lora-7b`, and runs a Gradio interface for inference on a specified input. Users should treat this as example code for the use of the model, and modify it as needed.

### If you want to finetune LLaMA, please use (`finetune.py`) for the original [Alpaca-LoRA](https://github.com/tloen/alpaca-lora) to train these models
### Otherwise, using ('train.py') for our BLOOM-LoRA

This file contains a straightforward application of PEFT to the LLaMA model,
as well as some code related to prompt construction and tokenization.
Near the top of this file is a set of hardcoded hyperparameters that you should feel free to modify.
PRs adapting this code to support larger models are always welcome.

### Checkpoint export (`export_*_checkpoint.py`)

These files contain scripts that merge the LoRA weights back into the base model
for export to Hugging Face format and to PyTorch `state_dicts`.
They should help users
who want to run inference in projects like [llama.cpp](https://github.com/ggerganov/llama.cpp)
or [alpaca.cpp](https://github.com/antimatter15/alpaca.cpp).

### Dataset

In addition to `alpaca_data.json`, which contains the original Stanford Alpaca dataset,
we also include `alpaca_data_cleaned.json`, which has been [stripped of various tokenization artifacts](https://github.com/tloen/alpaca-lora/pull/32)
with the help of @gururise.
This file is now used by default in the training script.

@AndriyMulyar has also provided interactive, embedding-based visualizations of the original dataset's [instructions](https://atlas.nomic.ai/map/alpaca_instructions)
and [outputs](https://atlas.nomic.ai/map/alpaca_outputs),
as well as [clusters of bad examples](https://atlas.nomic.ai/map/d2139cc3-bc1c-441c-8d6f-3e6ffbbc2eda/838019ff-8fe2-42ba-809a-d86d2b98cd50/-18.11668742841587/-11.348087116836096/-20.88850316347706/-17.680468640801223/774455612).

### Notes

- We can likely improve our model performance significantly if we combed through the data and fixed bad examples; in fact, dataset quality might be our bottleneck.
- We're continually fixing bugs and conducting training runs, and the weights on the Hugging Face Hub are being updated accordingly. In particular, those facing issues with response lengths should make sure that they have the latest version of the weights and code.

### Example outputs


**BLOOM-560M-LoRA**:








**Instruction**: Tell me about alpacas.

**BLOOM-7b1-LoRA**: Alpacas are native to the Andes Mountains of South America, where they have been domesticated for thousands of years. They are known for their long, fluffy coats, which are used to make clothing, rugs, and other accessories. Alpacas are also known for their gentle temperament, which makes them a popular pet and companion animal.

**Alpaca-LoRA**: Alpacas are members of the camelid family and are native to the Andes Mountains of South America. They are known for their soft, luxurious fleece, which is used to make clothing, blankets, and other items. Alpacas are herbivores and graze on grasses and other plants. They are social animals and live in herds of up to 20 individuals.

**Stanford Alpaca**: Alpacas are small, fluffy animals related to camels and llamas. They are native to Peru and Bolivia, and were first domesticated around 5,000 years ago. They are kept mainly for their fine, soft fleece, which is used to make knitwear and other garments. Alpacas are herd animals and live in small family groups, led by an older male. They are highly social animals and can form strong bonds with their owners.

**text-davinci-003**: Alpacas are a domesticated species of South American camelid, similar to llamas. They are native to the Andes Mountains in South America and are kept as livestock for their soft, luxurious wool. Alpacas are social animals and live in herds of up to 20 individuals. They are typically kept in small herds of two to five animals, and are relatively easy to care for. Alpacas are herbivores and feed on grass, hay, and other vegetation. They are also known for their gentle and friendly nature, making them popular as pets.

---

**Instruction**: Tell me about the president of Mexico in 2019.

**BLOOM-7b1-LoRA**: The president of Mexico in 2019 is Andrés Manuel López Obrador. He was elected in 2018 and took office in December of that year. López Obrador is known for his populist style of government and his support for social programs such as free health care and free education. He is also known for his opposition to the United States and its policies in Latin America. López Obrador has also been criticized for his lack of experience in government and his inability to govern effectively. Despite these criticisms, López Obrador has been able to gain the support of the majority of Mexicans and has been able to implement many of his policies.

**Alpaca-LoRA**: The president of Mexico in 2019 was Andrés Manuel López Obrador, who took office on December 1, 2018. He is a member of the National Regeneration Movement (MORENA) political party and is the first left-wing president of Mexico since 1946. He is known for his anti-corruption and anti-neoliberal policies, as well as his commitment to improving the living conditions of the Mexican people.

**Stanford Alpaca**: In 2019, Andres Manuel Lopez Obrador was sworn in as President of Mexico. He is the country’s 58th president and the first from the left-wing Morena party. He was elected in 2018 on a promise to combat corruption and reduce poverty and has faced criticism for his handling of the economy, crime and the country’s relationship with the United States. He is also known for his ambitious social welfare programs and has been praised for raising the minimum wage and providing aid to low-income families. He is seen as a champion of democracy and human rights in Mexico.

**text-davinci-003**: The president of Mexico in 2019 is Andrés Manuel López Obrador, commonly known as AMLO. He is a Mexican politician who has been the President of Mexico since December 1, 2018. He is a member of the National Regeneration Movement (MORENA) party and previously served as the Head of Government of Mexico City from 2000 to 2005. He is a left-wing populist and has promised to reduce corruption and inequality in Mexico. He has also promised to increase public spending on social programs and infrastructure projects.

---

**Instruction**: Tell me about the king of France in 2019.

**BLOOM-7b1-LoRA**: The king of France in 2019 is Emmanuel Macron. Macron was elected as the president of France in May 2017. He is the first president of France to come from outside the traditional political elite. Macron is a former banker and a member of the Socialist Party. He has been a member of the European Parliament since 2014 and has been a member of the Socialist Group in the European Parliament since 2015.

**Alpaca-LoRA**: The King of France in 2019 is Emmanuel Macron, who was elected in May 2017. He is the youngest president in the history of the Fifth Republic and the first president to be born after World War II. He is also the first president to have never held elected office before.

**Stanford Alpaca**: In 2019, the King of France was Emmanuel Macron. He was sworn in as President of France on May 14, 2017, and has been leading the country ever since. He is a member of the centrist La République En Marche! Party. He is known for his reformist agenda and has faced a number of protests from both the right and left of the political spectrum. He is the youngest French president in history.

**text-davinci-003**: The King of France is a largely symbolic role, as France is a republic. The current President of France is Emmanuel Macron, who was elected in 2017.

---

**Instruction**: List all Canadian provinces in alphabetical order.

**BLOOM-7b1-LoRA**: Alberta, British Columbia, Manitoba, New Brunswick, Newfoundland and Labrador, Nova Scotia, Northwest Territories, Nunavut, Ontario, Prince Edward Island, Quebec, Saskatchewan, Southwest Territories, Yukon, British Columbia, Alberta, Manitoba, New Brunswick, Newfoundland and Labrador, Nova Scotia, Northwest Territories, Nunavut, Ontario, Prince Edward Island, Quebec, Saskatchewan, Southwest Territories, Yukon, British Columbia, Alberta, Manitoba, New Brunswick, Newfoundland and Labrador, Nova Scotia, Northwest Territories, Nunavut, Ontario, Prince Edward Island, Quebec, Saskatchewan

**Alpaca-LoRA**: Alberta, British Columbia, Manitoba, New Brunswick, Newfoundland and Labrador, Nova Scotia, Ontario, Prince Edward Island, Quebec, Saskatchewan

**Stanford Alpaca**: Alberta, British Columbia, Manitoba, New Brunswick, Newfoundland and Labrador, Nova Scotia, Ontario, Prince Edward Island, Quebec, Saskatchewan, Northwest Territories, Nunavut, Yukon.

**text-davinci-003**:

1. Alberta
2. British Columbia
3. Manitoba
4. New Brunswick
5. Newfoundland and Labrador
6. Northwest Territories
7. Nova Scotia
8. Nunavut
9. Ontario
10. Prince Edward Island
11. Quebec
12. Saskatchewan
13. Yukon

---

**Instruction**: Write a Python program that prints the first 10 Fibonacci numbers.

**BLOOM-7b1-LoRA**:

```def fibonacci(n):
    if n == 0:
        return 0
    elif n == 1:
        return 1
    else:
        return fibonacci(n - 1) + fibonacci(n - 2)

for i in range(10):
    print(fibonacci(i))

# Output:
# 0
# 1
# 1
# 2
# 3
# 5
# 8
# 13
# 21
# 34
```

**Alpaca-LoRA**:

```python
def fibonacci(n):
    if n == 0:
        return 0
    elif n == 1:
        return 1
    else:
        return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(10))
```

**Stanford Alpaca**:

```python
numbers = [0, 1]

for i in range(2, 11):
    numbers.append(numbers[i-2] + numbers[i-1])

print(numbers)
# Output: [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

**text-davinci-003**:

```python
# Fibonacci numbers are the numbers in the following integer sequence, called the Fibonacci sequence, and characterized by the fact that every number after the first two is the sum of the two preceding ones.

# Fibonacci sequence: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, ...

# Program:

# Initialize the first two numbers of the Fibonacci sequence
a = 0
b = 1

# Print the first 10 numbers of the Fibonacci sequence
print(a)
print(b)

for i in range(2,10):
    c = a + b
    a = b
    b = c
    print(c)
```

---

**Instruction**: Write a program that prints the numbers from 1 to 100. But for multiples of three print 'Fizz' instead of the number and for the multiples of five print 'Buzz'. For numbers which are multiples of both three and five print 'FizzBuzz'.

**BLOOM-7b1-LoRA**: 
```
def print_numbers(num):
    if num % 3 == 0:
        print('Fizz')
    elif num % 5 == 0:
        print('Buzz')
    else:
        print(num)

print_numbers(1)
print_numbers(2)
print_numbers(3)
print_numbers(4)
print_numbers(5)
print_numbers(6)
print_numbers(7)
print_numbers(8)
print_numbers(9)
print_numbers(10)
print_numbers(11)
print_numbers(12)
print_numbers(13)
print_numbers(14)
```

**Alpaca-LoRA**:

```python
for i in range(1, 101):
    if i % 3 == 0 and i % 5 == 0:
        print("FizzBuzz")
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)
```

**Stanford Alpaca**:

```python
for i in range(1, 101):
    if i % 3 == 0 and i % 5 == 0:
        print('FizzBuzz')
    elif i % 3 == 0:
        print('Fizz')
    elif i % 5 == 0:
        print('Buzz')
    else:
        print(i)
```

**text-davinci-003**:

```javascript
for (let i = 1; i <= 100; i++) {
  if (i % 3 === 0 && i % 5 === 0) {
    console.log("FizzBuzz");
  } else if (i % 3 === 0) {
    console.log("Fizz");
  } else if (i % 5 === 0) {
    console.log("Buzz");
  } else {
    console.log(i);
  }
}
```

---

**Instruction**: Tell me five words that rhyme with 'shock'.

**BLOOM-7b1-LoRA**:
The five words that rhyme with "shock" are:
Shock, shock, shock, shock, shock

**Alpaca-LoRA**: Flock, lock, rock, stock, and sock.

**Stanford Alpaca**: Five words that rhyme with shock are: rock, pop, shock, cook, and snock.

**text-davinci-003**:

1. Block
2. Lock
3. Dock
4. Clock
5. Rock

---

**Instruction**: Translate the sentence 'I have no mouth but I must scream' into Spanish.

**BLOOM-7b1-LoRA**: Yo no tengo boca pero debo gritar

**Alpaca-LoRA**: No tengo boca pero tengo que gritar.

**Stanford Alpaca**: No tengo boca, pero debo gritar.

**text-davinci-003**: No tengo boca pero debo gritar.
