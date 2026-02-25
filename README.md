# AI-Tutor
AI 와 함께 학습하는 수학공부
import streamlit as st
import sympy as sp
import openai

# --- 설정 및 API 연결 ---
st.set_page_config(page_title="1차방정식 AI 튜터", layout="wide")
client = openai.OpenAI(api_key="YOUR_OPENAI_API_KEY")

# --- 세션 상태 초기화 (대화 기록 저장) ---
if "messages" not in st.session_state:
    st.session_state.messages = []
if "current_eq" not in st.session_state:
    st.session_state.current_eq = "2*x + 4 = 10"

# --- 사이드바: 문제 설정 ---
with st.sidebar:
    st.title("🔢 학습 설정")
    new_eq = st.text_input("연습할 방정식을 입력하세요:", st.session_state.current_eq)
    if st.button("문제 업데이트"):
        st.session_state.current_eq = new_eq
        st.session_state.messages = [] # 대화 초기화
        st.rerun()
    st.info("입력 예시: 3*x - 5 = 10")

# --- 메인 화면: AI 튜터와 대화 ---
st.title("🤖 나만의 AI 수학 선생님")
st.subheader(f"현재 풀고 있는 문제: :blue[{st.session_state.current_eq}]")

# 기존 대화 표시
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# 학생의 입력 (Chat Input)
if prompt := st.chat_input("풀이 과정이나 질문을 입력하세요 (예: 2x = 6)"):
    # 1. 학생 입력 화면에 표시
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user"):
        st.markdown(prompt)

    # 2. SymPy 검증 (간이 로직)
    is_correct = "알 수 없음"
    try:
        # 입력값 전처리 (2x -> 2*x)
        processed_prompt = prompt.replace("x", "*x").replace("**x", "*x")
        if "=" in processed_prompt:
            x = sp.symbols('x')
            # 정답 해 구하기
            l_orig, r_orig = st.session_state.current_eq.split('=')
            sol_orig = sp.solve(sp.Eq(sp.simplify(l_orig), sp.simplify(r_orig)), x)
            # 학생 단계 해 구하기
            l_step, r_step = processed_prompt.split('=')
            sol_step = sp.solve(sp.Eq(sp.simplify(l_step), sp.simplify(r_step)), x)
            is_correct = "정답" if sol_orig == sol_step else "오답"
    except:
        is_correct = "형식 오류"

    # 3. AI 피드백 생성
    with st.chat_message("assistant"):
        ai_instruction = f"""
        문제: {st.session_state.current_eq}
        학생의 단계: {prompt}
        검증결과: {is_correct}
        수학 선생님으로서 학생에게 짧고 친절하게 피드백을 줘.
        """
        
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "system", "content": "친절한 수학 튜터."},
                      {"role": "user", "content": ai_instruction}]
        )
        msg = response.choices[0].message.content
        st.markdown(msg)
        st.session_state.messages.append({"role": "assistant", "content": msg})
