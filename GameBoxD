import streamlit as st
import requests
import json
import os
from datetime import datetime

# ---------------- CONFIG ----------------
RAWG_API_KEY = os.getenv("RAWG_API_KEY") or "4193f734e2f1493cb033d60b1363edfe"
BASE_URL = "https://api.rawg.io/api"
DATA_FILE = "user_games.json"

st.set_page_config(page_title="GameBoxd", layout="wide")

# ---------------- STORAGE ----------------

def load_data():
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    return {}


def save_data(data):
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)


user_data = load_data()

# ---------------- API ----------------

def search_games(query="", page_size=20):
    params = {
        "key": RAWG_API_KEY,
        "search": query,
        "page_size": page_size
    }
    r = requests.get(f"{BASE_URL}/games", params=params)
    return r.json().get("results", [])


def get_game_details(game_id):
    params = {"key": RAWG_API_KEY}
    r = requests.get(f"{BASE_URL}/games/{game_id}", params=params)
    return r.json()


# ---------------- UI HELPERS ----------------

def game_card(game):
    col = st.container()
    with col:
        st.image(game.get("background_image"), use_container_width=True)
        st.markdown(f"**{game['name']}**")
        if st.button("Ver detalhes", key=f"btn_{game['id']}"):
            st.session_state["selected_game"] = game["id"]


# ---------------- SIDEBAR ----------------
st.sidebar.title("🎮 GameBoxd")

filter_view = st.sidebar.radio(
    "Filtrar por status",
    ["Todos", "Quero Jogar", "Jogando", "Já Joguei"]
)

# ---------------- MAIN ----------------
st.title("GameBoxd – Seu Letterboxd de Games")

search = st.text_input("🔍 Buscar jogos", placeholder="Digite o nome do jogo...")

if "selected_game" not in st.session_state:
    st.session_state.selected_game = None

# ---------------- GAME LIST ----------------

games = search_games(search)

cols = st.columns(5)
idx = 0

for game in games:
    game_id = str(game["id"])

    # Filter by status
    if filter_view != "Todos":
        if game_id not in user_data:
            continue
        if user_data[game_id].get("status") != filter_view:
            continue

    with cols[idx % 5]:
        game_card(game)
    idx += 1

# ---------------- POPUP / DETAILS ----------------

if st.session_state.selected_game:
    game_id = st.session_state.selected_game
    details = get_game_details(game_id)

    st.divider()
    st.subheader(details.get("name"))

    cols = st.columns([1, 2])

    with cols[0]:
        st.image(details.get("background_image"), use_container_width=True)

    with cols[1]:
        st.markdown(details.get("description_raw", "Sem descrição disponível."))
        st.markdown(f"**Lançamento:** {details.get('released')} ")
        st.markdown(f"**Metacritic:** {details.get('metacritic')} ")

        # User interaction
        game_key = str(game_id)
        if game_key not in user_data:
            user_data[game_key] = {
                "status": "Quero Jogar",
                "rating": 0,
                "updated": str(datetime.now())
            }

        status = st.selectbox(
            "Status",
            ["Quero Jogar", "Jogando", "Já Joguei"],
            index=["Quero Jogar", "Jogando", "Já Joguei"].index(user_data[game_key]["status"])
        )

        rating = st.slider("Avaliação", 0, 10, int(user_data[game_key]["rating"]))

        if st.button("Salvar"):
            user_data[game_key]["status"] = status
            user_data[game_key]["rating"] = rating
            user_data[game_key]["updated"] = str(datetime.now())
            save_data(user_data)
            st.success("Salvo com sucesso!")

        if st.button("Fechar"):
            st.session_state.selected_game = None
            st.experimental_rerun()

# ---------------- FOOTER ----------------
st.caption("Powered by RAWG API | Projeto pessoal")
