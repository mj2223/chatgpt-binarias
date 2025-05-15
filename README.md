# chatgpt-binarias
Análise de opções binárias com IA.
from fastapi import FastAPI, Request
from pydantic import BaseModel
import openai
import os

# Configure sua API Key da OpenAI
openai.api_key = "SUA_API_KEY_AQUI"

app = FastAPI()

# Modelo de entrada
class CandleData(BaseModel):
    par: str
    timeframe: str
    candles: list  # lista de velas no formato [[o,h,l,c], [o,h,l,c], ...]

@app.post("/analise")
async def analisar_candles(data: CandleData):
    # Construir prompt para o ChatGPT
    candles_str = "\n".join([str(c) for c in data.candles])
    
    prompt = f"""
Você é um analista técnico profissional.
Analise o par de moedas {data.par}, timeframe {data.timeframe}.
As últimas velas estão no formato [open, high, low, close]:

{candles_str}

Baseado nessas velas, existe alguma oportunidade de CALL ou PUT? Responda com base em price action e padrão de velas.
"""
    try:
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=[
                {"role": "system", "content": "Você é um analista técnico de mercado financeiro especializado em price action."},
                {"role": "user", "content": prompt}
            ]
        )
        return {"analise": response['choices'][0]['message']['content']}
    except Exception as e:
        return {"erro": str(e)}
