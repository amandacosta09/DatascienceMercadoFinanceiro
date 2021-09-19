---
title: "Redes Neurais Profundas"
---
# Conceito (O que é? Pra que serve? )
É uma sub-área da aprendizagem de máquina que emprega algoritmos para processar dados e imitar o processamento feito pelo cérebro humano. 
Deep learning usa cadamas de neurônios matemáticos para processar dados, compreender a fala, reconhecer objetos em imagens, reproduzir objetos e até reproduzir a voz de pessoas.

# Classes de Problemas com melhores resultados
É melhor utilizada para problemas de maiores complexidades, como por exemplo análise de imagens, análise de audios, detecção de fraudes, sistemas de recomendação, previsão de falhas, entre outros.

# Definição Teórica e Modelagem Matemática
Em redes neurais profundas criamos um modelo de várias camadas de neurônios. A criação de camadas e a quantidade de neurônios em cada camada é definida de acordo com a necessidade do projeto.
Cada camada irá abstrair e criar uma representação de uma parte do problema, e assim quando combinadas as camadas é possível dizer um resultado com uma boa acurácia.Por exemplo em um problema de reconhecimento de imagens teremos na primeira camada um input de uma matriz de pixels; na segunda camada podemos ter uma composição e rearranjo de pontas; na terceira camada pode conter o processo de encontrar um parte do objeto que se deseja reconhecer e na quarta camada o objeto a que se deseja reconhecer.

# Vantagens e Desvantagens (limitações)
Vantagens:
- As features são ajustadas automaticamente para melhorar a saída;
- É robusto a variações de dados (aprendido automaticamente);
- A mesma rede neural pode ser usada para diferentes aplicações e tipos de dados;
- É possível ter um paralelismo (multiplas GPUs) e por isso entrega melhor performance em grandes volumes de dados;
- É flexível para ser adaptada a novos problemas.

Desvantagens:
- Requere um grande volume de dados para performar melhor que outra técnicas;
- É extremamente cara para treinar devido ao modelo complexo e requere hardwares caros como GPUs e várias máquinas;
- É de difícil modelagem para quem tem pouca experiência na área.

# Exemplo de uma aplicação em Python



{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Generate restaurant reviews using deep learning\n",
    "\n",
    "This Code Pattern will guide you through installing Keras and Tensorflow, downloading data of Yelp reviews and training a language model using recurrent neural networks, or RNNs, to generate text.\n",
    "\n",
    "The original Yelp data used in this Code Pattern can be found on [Kaggle](https://www.kaggle.com/c/yelp-recruiting/data) and as a [zip file](https://github.com/IBM/deep-learning-language-model/blob/master/yelp_test_set.zip) in the GitHub repository.\n",
    "\n",
    "\n",
    "<a id=\"section0\"></a>\n",
    "### 0. Prerequisites\n",
    "\n",
    "Before you run this notebook complete the following steps:\n",
    "- Insert a project credential\n",
    "- Load required files\n",
    "- Import required packages\n",
    "\n",
    "#### Insert a project credential\n",
    "\n",
    "When you import this project from the Watson Studio Gallery, a token should be automatically generated and inserted at the top of this notebook as a code cell such as the one below:\n",
    "\n",
    "```python\n",
    "# @hidden_cell\n",
    "# The following code contains the credentials for a file in your IBM Cloud Object Storage.\n",
    "# You might want to remove those credentials before you share your notebook.\n",
    "credentials = {\n",
    "    'IBM_API_KEY_ID': '*******************************',\n",
    "    'IAM_SERVICE_ID': '*******************************',\n",
    "    'ENDPOINT': '*******************************',\n",
    "    'IBM_AUTH_ENDPOINT': '*******************************',\n",
    "    'BUCKET': '*******************************',\n",
    "    'FILE': 'FILENAME.csv'\n",
    "}\n",
    "```\n",
    "\n",
    "If you do not see the cell above, follow these steps to enable the notebook to access the dataset from the project's resources:\n",
    "\n",
    "* Click on `More -> Insert project token` in the top-right menu section\n",
    "\n",
    "![ws-credential.mov](https://media.giphy.com/media/iFy3ERX0PHsOpTgogb/giphy.gif)\n",
    "\n",
    "* This should insert a cell at the top of this notebook similar to the example given above.\n",
    "\n",
    "  > If an error is displayed indicating that no project token is defined, follow [these instructions](https://dataplatform.cloud.ibm.com/docs/content/wsj/analyze-data/token.html?audience=wdp&context=data).\n",
    "\n",
    "* Run the newly inserted cell before proceeding with the notebook execution below\n",
    "\n",
    "\n",
    "#### Import required packages\n",
    "\n",
    "Import and configure the required packages."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "from ibm_botocore.client import Config\n",
    "import ibm_boto3"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "def download_file_cos(credentials, local_file_name, key): \n",
    "    '''\n",
    "    Wrapper function to download a file from cloud object storage using the\n",
    "    credential dict provided and loading it into memory\n",
    "    '''\n",
    "    cos = ibm_boto3.client(service_name='s3',\n",
    "    ibm_api_key_id=credentials['IBM_API_KEY_ID'],\n",
    "    ibm_service_instance_id=credentials['IAM_SERVICE_ID'],\n",
    "    ibm_auth_endpoint=credentials['IBM_AUTH_ENDPOINT'],\n",
    "    config=Config(signature_version='oauth'),\n",
    "    endpoint_url=credentials['ENDPOINT'])\n",
    "    try:\n",
    "        res=cos.download_file(Bucket=credentials['BUCKET'],Key=key,Filename=local_file_name)\n",
    "    except Exception as e:\n",
    "        print(Exception, e)\n",
    "    else:\n",
    "        print('File Downloaded')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "# @hidden_cell\n",
    "# The following code contains the credentials for a file in your IBM Cloud Object Storage.\n",
    "# You might want to remove those credentials before you share your notebook.\n",
    "credentials = {\n",
    "    'IBM_API_KEY_ID': '*******************************',\n",
    "    'IAM_SERVICE_ID': '*******************************',\n",
    "    'ENDPOINT': '*******************************',\n",
    "    'IBM_AUTH_ENDPOINT': '*******************************',\n",
    "    'BUCKET': '*******************************',\n",
    "    'FILE': 'char_indices.txt'\n",
    "}"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "download_file_cos(credentials,'char_indices.txt','char_indices.txt')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "# @hidden_cell\n",
    "# The following code contains the credentials for a file in your IBM Cloud Object Storage.\n",
    "# You might want to remove those credentials before you share your notebook.\n",
    "credentials_1 = {\n",
    "    'IBM_API_KEY_ID': '*******************************',\n",
    "    'IAM_SERVICE_ID': '*******************************',\n",
    "    'ENDPOINT': '*******************************',\n",
    "    'IBM_AUTH_ENDPOINT': '*******************************',\n",
    "    'BUCKET': '*******************************',\n",
    "    'FILE': 'indices_char.txt'\n",
    "}"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "download_file_cos(credentials_1,'indices_char.txt','indices_char.txt')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "# @hidden_cell\n",
    "# The following code contains the credentials for a file in your IBM Cloud Object Storage.\n",
    "# You might want to remove those credentials before you share your notebook.\n",
    "credentials_2 = {\n",
    "    'IBM_API_KEY_ID': '*******************************',\n",
    "    'IAM_SERVICE_ID': '*******************************',\n",
    "    'ENDPOINT': '*******************************',\n",
    "    'IBM_AUTH_ENDPOINT': '*******************************',\n",
    "    'BUCKET': '*******************************',\n",
    "    'FILE': 'yelp_100_3.txt'\n",
    "}"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "download_file_cos(credentials_2,'yelp_100_3.txt','yelp_100_3.txt')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "# @hidden_cell\n",
    "# The following code contains the credentials for a file in your IBM Cloud Object Storage.\n",
    "# You might want to remove those credentials before you share your notebook.\n",
    "credentials_3 = {\n",
    "    'IBM_API_KEY_ID': '*******************************',\n",
    "    'IAM_SERVICE_ID': '*******************************',\n",
    "    'ENDPOINT': '*******************************',\n",
    "    'IBM_AUTH_ENDPOINT': '*******************************',\n",
    "    'BUCKET': '*******************************',\n",
    "    'FILE': 'transfer_weights'\n",
    "}"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "download_file_cos(credentials_3,'transfer_weights','transfer_weights')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Make sure the all four files are downloaded correctly\n",
    "!ls"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## Execute transfer_learn.py to run the model\n",
    "#### Analyze the notebook below to help understand what transfer_learn.py is doing."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "To help understand what we're doing here, below we use a keras sequential model of the LSTM variety, mentioned in the README of this repository. We use this variety of RNN model so that we can include hidden layers of memory to generate more accurate text. We then use the Adam optimizer with categorical_crossentropy and begin by loading our transfer_weights. We define the sample with a temperature of 0.6. The temperature here is a parameter than can be used in the softmax function which controls the level of newness generated where 1 constricts sampling and leads to less diverse/more repetitive text and 0 has completely diverse text. In this case we are leaning slightly more towards repetition though in this particular model, we generate multiple diversities which we can compare to one another to see which works best for us. We then train the model and save our weights into transfer_weights. After executing you should see tensorflow start up and then various epochs running on your screen followed by generated text."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "from __future__ import print_function\n",
    "import json\n",
    "from keras.models import Sequential\n",
    "from keras.layers import Dense, Activation\n",
    "from keras.layers import LSTM\n",
    "from keras.optimizers import Adam\n",
    "from keras.utils.data_utils import get_file\n",
    "import numpy as np\n",
    "import random\n",
    "import sys\n",
    "import requests\n",
    "import pandas as pd"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "path = 'yelp_100_3.txt'\n",
    "text = open(path).read().lower()\n",
    "print('corpus length:', len(text))\n",
    "char_indices = json.loads(open('char_indices.txt').read())\n",
    "indices_char = json.loads(open('indices_char.txt').read())\n",
    "chars = sorted(char_indices.keys())\n",
    "print(indices_char)\n",
    "#chars = sorted(list(set(text)))\n",
    "print('total chars:', len(chars))\n",
    "#char_indices = dict((c, i) for i, c in enumerate(chars))\n",
    "#indices_char = dict((i, c) for i, c in enumerate(chars))\n",
    "\n",
    "# cut the text in semi-redundant sequences of maxlen characters\n",
    "maxlen = 256\n",
    "step = 3\n",
    "sentences = []\n",
    "next_chars = []\n",
    "for i in range(0, len(text) - maxlen, step):\n",
    "    sentences.append(text[i: i + maxlen])\n",
    "    next_chars.append(text[i + maxlen])\n",
    "print('nb sequences:', len(sentences))\n",
    "\n",
    "print('Vectorization...')\n",
    "X = np.zeros((len(sentences), maxlen, len(chars)), dtype=np.bool)\n",
    "y = np.zeros((len(sentences), len(chars)), dtype=np.bool)\n",
    "for i, sentence in enumerate(sentences):\n",
    "    for t, char in enumerate(sentence):\n",
    "        X[i, t, char_indices[char]] = 1\n",
    "    y[i, char_indices[next_chars[i]]] = 1\n",
    "\n",
    "\n",
    "# build the model: a single LSTM\n",
    "print('Build model...')\n",
    "model = Sequential()\n",
    "model.add(LSTM(1024, return_sequences=True, input_shape=(maxlen, len(chars))))\n",
    "model.add(LSTM(512, return_sequences=False))\n",
    "model.add(Dense(len(chars)))\n",
    "model.add(Activation('softmax'))\n",
    "\n",
    "optimizer = Adam(lr=0.002)\n",
    "model.compile(loss='categorical_crossentropy', optimizer=optimizer)\n",
    "\n",
    "model.load_weights(\"transfer_weights\")\n",
    "\n",
    "def sample(preds, temperature=.6):\n",
    "    # helper function to sample an index from a probability array\n",
    "    preds = np.asarray(preds).astype('float64')\n",
    "    preds = np.log(preds) / temperature\n",
    "    exp_preds = np.exp(preds)\n",
    "    preds = exp_preds / np.sum(exp_preds)\n",
    "    probas = np.random.multinomial(1, preds, 1)\n",
    "    return np.argmax(probas)\n",
    "\n",
    "# train the model, output generated text after each iteration\n",
    "for iteration in range(1, 60):\n",
    "    print()\n",
    "    print('-' * 50)\n",
    "    print('Iteration', iteration)\n",
    "    x = np.zeros((1, maxlen, len(chars)))\n",
    "    preds = model.predict(x, verbose=0)[0]\n",
    "    \n",
    "    model.fit(X, y, batch_size=128, epochs=1)\n",
    "\n",
    "    start_index = random.randint(0, len(text) - maxlen - 1)\n",
    "    #start_index = char_indices[\"{\"]\n",
    "\n",
    "    for diversity in [0.2, 0.4, 0.6, 0.8]:\n",
    "        print()\n",
    "        print('----- diversity:', diversity)\n",
    "\n",
    "        generated = ''\n",
    "        sentence = text[start_index: start_index + maxlen]\n",
    "        generated += sentence\n",
    "        print('----- Generating with seed: \"' + sentence + '\"')\n",
    "        sys.stdout.write(generated)\n",
    "        for i in range(400):\n",
    "            x = np.zeros((1, maxlen, len(chars)))\n",
    "            for t, char in enumerate(sentence):\n",
    "                x[0, t, char_indices[char]] = 1.\n",
    "\n",
    "            preds = model.predict(x, verbose=0)[0]\n",
    "            next_index = sample(preds, diversity)\n",
    "            #print(next_index)\n",
    "            #print (indices_char)\n",
    "            next_char = indices_char[str(next_index)]\n",
    "\n",
    "            generated += next_char\n",
    "            sentence = sentence[1:] + next_char\n",
    "\n",
    "            sys.stdout.write(next_char)\n",
    "            sys.stdout.flush()\n",
    "        print()\n",
    "model.save_weights(\"transfer_weights\")"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3.6",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.6.9"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 2
}

