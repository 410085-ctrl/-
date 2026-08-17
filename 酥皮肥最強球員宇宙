import streamlit as st
import google.generativeai as genai
import pandas as pd
import json
from PIL import Image

st.set_page_config(page_title="酥皮肥宇宙 GOAT 排行榜", page_icon="🏆", layout="wide")

st.title("🏆 酥皮肥宇宙：GOAT 歷史地位自動評分站")
st.write("只要上傳「酥皮肥」球員生涯海報截圖，系統會自動透過 AI 辨識數據、計算 GOAT 評分並生成排行榜！")

# 側邊欄設定
st.sidebar.header("⚙️ 設定與 API Key")
api_key = st.sidebar.text_input("請輸入 Google Gemini API Key", type="password")
st.sidebar.markdown("[👉 免費獲取 Gemini API Key](https://aistudio.google.com/)")

uploaded_files = st.file_uploader("上傳球員海報截圖（可一次選擇多張）", type=["png", "jpg", "jpeg"], accept_multiple_files=True)

def parse_card(image, key):
    genai.configure(api_key=key)
    model = genai.GenerativeModel('gemini-2.5-flash')
    
    prompt = """
    請仔細分析這張 NBA 模擬球員海報截圖，並嚴格回傳一個 JSON 物件，不要包含 Markdown 格式外的雜文。
    格式如下：
    {
        "name": "球員名稱",
        "position": "位置 (如: PG, SG, SF, PF, C)",
        "era": "活躍年份 (如: 1984-2004)",
        "rings": 總冠軍數(整數),
        "fmvp": FMVP數(整數),
        "mvp": MVP數(整數),
        "dpoy": DPOY數(整數),
        "scoring_titles": 得分王次數(整數),
        "assist_titles": 助攻王次數(整數),
        "rebound_titles": 籃板王次數(整數),
        "total_points": 生涯總得分(整數)
    }
    """
    res = model.generate_content([prompt, image])
    text = res.text.replace("```json", "").replace("```", "").strip()
    return json.loads(text)

def calc_score(d):
    score = (
        d.get('rings', 0) * 10 +
        d.get('fmvp', 0) * 8 +
        d.get('mvp', 0) * 7 +
        d.get('dpoy', 0) * 4 +
        d.get('scoring_titles', 0) * 3 +
        d.get('assist_titles', 0) * 2 +
        d.get('rebound_titles', 0) * 2 +
        (d.get('total_points', 0) / 5000)
    )
    return round(score, 2)

if uploaded_files:
    if not api_key:
        st.warning("請先在左側欄位輸入你的 Gemini API Key！")
    else:
        players = []
        with st.spinner("AI 正在解析卡片數據並計算排名中..."):
            for file in uploaded_files:
                try:
                    img = Image.open(file)
                    data = parse_card(img, api_key)
                    data['filename'] = file.name
                    data['goat_score'] = calc_score(data)
                    players.append(data)
                except Exception as e:
                    st.error(f"解析 {file.name} 失敗: {e}")
        
        if players:
            df = pd.DataFrame(players)
            df = df.sort_values(by="goat_score", ascending=False).reset_index(drop=True)
            df.index += 1
            
            st.subheader("📊 終極 GOAT 排行榜")
            st.dataframe(
                df[["name", "position", "era", "rings", "fmvp", "mvp", "dpoy", "scoring_titles", "total_points", "goat_score"]],
                column_config={
                    "name": "球員",
                    "position": "位置",
                    "era": "時代",
                    "rings": "總冠軍",
                    "fmvp": "FMVP",
                    "mvp": "MVP",
                    "dpoy": "DPOY",
                    "scoring_titles": "得分王",
                    "total_points": "總得分",
                    "goat_score": "GOAT 綜合評分"
                },
                use_container_width=True
            )
