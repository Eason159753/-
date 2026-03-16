# -import streamlit as st
import pandas as pd
from datetime import datetime
import os

# 設定網頁標題
st.set_page_config(page_title="失物招領系統", layout="centered")

st.title("📱 失物招領雲端系統")
st.write("手機拍照、電腦管理，通通搞定！")

# 建立儲存圖片的資料夾
if not os.path.exists("lost_images"):
    os.makedirs("lost_images")

# 初始化 Session State (模擬資料庫)
if 'data' not in st.session_state:
    st.session_state.data = []

# --- 側邊欄：登記新失物 ---
st.sidebar.header("📝 登記新物件")
with st.sidebar.form("input_form", clear_on_submit=True):
    name = st.text_input("物品名稱")
    loc = st.text_input("拾獲地點")
    
    # 核心功能：拍照或上傳
    img_file = st.camera_input("拍照或上傳照片")
    
    submitted = st.form_submit_button("提交登記")
    
    if submitted and name and loc:
        # 儲存圖片
        img_path = "無照片"
        if img_file:
            img_path = f"lost_images/{datetime.now().strftime('%Y%m%d%H%M%S')}.jpg"
            with open(img_path, "wb") as f:
                f.write(img_file.getbuffer())
        
        # 存入資料
        new_entry = {
            "時間": datetime.now().strftime("%Y-%m-%d %H:%M"),
            "物品名稱": name,
            "地點": loc,
            "照片路徑": img_path
        }
        st.session_state.data.append(new_entry)
        st.sidebar.success("登記成功！")

# --- 主畫面：顯示清單 ---
st.subheader("🔍 目前失物清單")

if not st.session_state.data:
    st.info("目前沒有紀錄。")
else:
    # 轉換成表格顯示
    df = pd.DataFrame(st.session_state.data)
    
    # 用卡片式佈局顯示內容
    for index, item in enumerate(st.session_state.data):
        with st.expander(f"📦 {item['物品名稱']} (於 {item['地點']})"):
            col1, col2 = st.columns([1, 2])
            with col1:
                if item['照片路徑'] != "無照片":
                    st.image(item['照片路徑'], use_column_width=True)
                else:
                    st.write("📷 無照片")
            with col2:
                st.write(f"**登記時間:** {item['時間']}")
                st.write(f"**拾獲地點:** {item['地點']}")
                if st.button(f"確認領取 (刪除編號 {index})", key=index):
                    st.session_state.data.pop(index)
                    st.rerun()
                    
