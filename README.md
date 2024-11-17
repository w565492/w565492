import tkinter as tk
from tkinter import scrolledtext
from tkinter import messagebox
from anthropic import Anthropic

# 直接在代码中设置API密钥（不推荐用于生产环境）
XAI_API_KEY = "输入你的grok的key"

# 创建Anthropic客户端
client = Anthropic(
    api_key=XAI_API_KEY,
    base_url="https://api.x.ai",
)

# 定义与GroK交互的函数
def ask_grok(question):
    try:
        response = client.messages.create(
            model="grok-beta",
            max_tokens=4096,
            system="You are Grok, a chatbot inspired by the Hitchhiker's Guide to the Galaxy.",
            messages=[
                {
                    "role": "user",
                    "content": question,
                },
            ],
        )
        return response.content
    except Exception as e:
        return f"An error occurred: {e}"

# 创建GUI
def create_gui():
    def on_send_click():
        user_message = user_input.get()
        if user_message.strip() == '':
            return

        # 在聊天框中显示用户消息
        chat_box.insert(tk.END, f"至简: {user_message}\n")

        # 清空输入框
        user_input.delete(0, tk.END)

        # 获取Grok的回应并显示
        grok_reply = ask_grok(user_message)
        chat_box.insert(tk.END, f"Grok: {grok_reply}\n\n")

        # 自动滚动到最新消息
        chat_box.yview(tk.END)

    def copy_text():
        selected_text = chat_box.get(tk.SEL_FIRST, tk.SEL_LAST)
        if selected_text:
            root.clipboard_clear()
            root.clipboard_append(selected_text)
            messagebox.showinfo("复制成功", "已将所选内容复制到剪贴板！")
        else:
            messagebox.showwarning("未选择内容", "请先选择要复制的文本内容！")

    # 创建主窗口
    root = tk.Tk()
    root.title("至简AI")
    root.geometry("500x600")  # 窗口大小

    # 创建Grid布局并设置行和列的权重，使其居中
    root.grid_rowconfigure(0, weight=1)
    root.grid_rowconfigure(1, weight=0)
    root.grid_rowconfigure(2, weight=0)
    root.grid_columnconfigure(0, weight=1)

    # 创建聊天显示框（使用scrollable text widget）
    chat_box = scrolledtext.ScrolledText(root, wrap=tk.WORD, width=60, height=20, state='normal', font=("Arial", 12))
    chat_box.grid(row=0, column=0, padx=10, pady=10, sticky="nsew")

    # 创建用户输入框
    user_input = tk.Entry(root, width=50, font=("Arial", 12))
    user_input.grid(row=1, column=0, padx=10, pady=10, sticky="nsew")

    # 创建发送按钮，并设置背景色为黄色
    send_button = tk.Button(root, text="发送", font=("Arial", 12), command=on_send_click, bg="yellow")
    send_button.grid(row=2, column=0, pady=10, sticky="nsew")

    # 创建复制按钮
    copy_button = tk.Button(root, text="复制", font=("Arial", 12), command=copy_text)
    copy_button.grid(row=3, column=0, pady=10, sticky="nsew")

    # 启动GUI主循环
    root.mainloop()

# 主函数
if __name__ == "__main__":
    create_gui()
