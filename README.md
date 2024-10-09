import tkinter as tk
from tkinter import messagebox, simpledialog, filedialog
import json
import os
import pandas as pd

# Đường dẫn file câu hỏi
QUESTION_FILE = 'questions.json'

# Khởi tạo các biến
score = 0
current_question = 0
questions = []
user_answers = {}

# Hàm để tải câu hỏi từ file JSON
def load_questions():
    global questions
    if os.path.exists(QUESTION_FILE):
        with open(QUESTION_FILE, 'r') as file:
            questions = json.load(file)
    else:
        questions = []

# Hàm để lưu câu hỏi vào file JSON
def save_questions():
    with open(QUESTION_FILE, 'w') as file:
        json.dump(questions, file)

# Hàm để kiểm tra câu trả lời của người dùng
def check_answer(selected_option):
    global user_answers, current_question
    user_answers[current_question] = selected_option
    update_question_buttons()
    
    for i in range(4):
        options[i].config(bg="light green" if i == selected_option else "white")

    # Đánh dấu câu trả lời đúng
    correct_answer = questions[current_question]['correct_answer']
    options[correct_answer].config(bg="light coral")

    update_status()

# Hàm để hiển thị câu hỏi lên giao diện
def load_question():
    question_label.config(text=questions[current_question]['question'])
    for i in range(4):
        options[i].config(text=questions[current_question]['answers'][i], bg="white")

    if current_question in user_answers:
        selected_option = user_answers[current_question]
        options[selected_option].config(bg="light green")  # Đánh dấu câu đã chọn
        options[questions[current_question]['correct_answer']].config(bg="light coral")  # Đánh dấu câu đúng
    else:
        for opt in options:
            opt.config(bg="white")  # Xóa đánh dấu

    update_question_buttons()  # Cập nhật màu của các nút câu hỏi

# Hàm để cập nhật trạng thái (câu đã trả lời, chưa trả lời)
def update_status():
    answered = len(user_answers)
    total = len(questions)
    unanswered = total - answered
    status_label.config(text=f"Tổng: {total} | Đã trả lời: {answered} | Chưa trả lời: {unanswered}")

# Hàm cập nhật bảng hiển thị trạng thái câu hỏi
def update_question_buttons():
    for i in range(len(questions)):
        if i in user_answers:
            question_buttons[i].config(bg="red")  # Đã trả lời
        else:
            question_buttons[i].config(bg="white")  # Chưa trả lời

    # Đánh dấu câu hỏi hiện tại
    for btn in question_buttons:
        btn.config(bg="white")  # Đặt lại màu cho tất cả
    question_buttons[current_question].config(bg="yellow")  # Đánh dấu câu hỏi hiện tại

# Hàm hiển thị kết quả cuối cùng
def show_result():
    for i in range(len(questions)):
        if i in user_answers and user_answers[i] == questions[i]['correct_answer']:
            global score
            score += 1
    messagebox.showinfo("Kết quả", f"Điểm số của bạn: {score}/{len(questions)}")
    window.quit()

# Hàm điều hướng câu tiếp theo
def next_question():
    global current_question
    if current_question < len(questions) - 1:
        current_question += 1
        load_question()

# Hàm điều hướng câu trước
def previous_question():
    global current_question
    if current_question > 0:
        current_question -= 1
        load_question()

# Hàm để thêm câu hỏi mới
def add_question():
    question = simpledialog.askstring("Thêm Câu Hỏi", "Nhập câu hỏi:")
    if not question:
        return
    
    answers = []
    for i in range(4):
        answer = simpledialog.askstring("Thêm Đáp Án", f"Nhập phương án {i + 1}:")
        if not answer:
            return
        answers.append(answer)
    
    correct_answer = simpledialog.askinteger("Đáp Án Đúng", "Nhập đáp án đúng (1-4):")
    if correct_answer not in [1, 2, 3, 4]:
        messagebox.showerror("Lỗi", "Đáp án đúng phải nằm trong khoảng từ 1 đến 4.")
        return

    new_question = {
        'question': question,
        'answers': answers,
        'correct_answer': correct_answer - 1
    }
    questions.append(new_question)
    save_questions()

    messagebox.showinfo("Thành Công", "Câu hỏi đã được thêm thành công!")
    if len(questions) == 1:
        load_question()

# Hàm để nhập câu hỏi từ file Excel
def import_questions():
    global questions
    file_path = filedialog.askopenfilename(filetypes=[("Excel Files", "*.xlsx")])
    if not file_path:
        return

    try:
        df = pd.read_excel(file_path)

        # Xóa câu hỏi trùng lặp
        unique_questions = []
        for index, row in df.iterrows():
            question = row['Câu hỏi']
            answers = [row['Phương án 1'], row['Phương án 2'], row['Phương án 3'], row['Phương án 4']]
            correct_answer = row['Đáp Án'] - 1  # Chỉ số bắt đầu từ 0
            
            # Kiểm tra xem câu hỏi đã tồn tại chưa
            if not any(q['question'] == question for q in unique_questions):
                new_question = {
                    'question': question,
                    'answers': answers,
                    'correct_answer': correct_answer
                }
                unique_questions.append(new_question)

        # Cập nhật danh sách câu hỏi
        questions.extend(unique_questions)
        save_questions()
        messagebox.showinfo("Thành Công", f"Đã nhập {len(unique_questions)} câu hỏi thành công!")
        
        # Hiển thị câu hỏi đầu tiên nếu có
        if len(questions) == len(unique_questions):
            load_question()
        update_question_buttons()
    except Exception as e:
        messagebox.showerror("Lỗi", f"Không thể nhập câu hỏi: {e}")

# Tạo giao diện
window = tk.Tk()
window.title("Ứng Dụng Trắc Nghiệm Nâng Cao")
window.geometry("800x600")
window.config(bg="#f0f0f0")

# Nhãn hiển thị câu hỏi
question_label = tk.Label(window, text="", font=("Arial", 18), wraplength=600, bg="#f0f0f0")
question_label.pack(pady=20)

# Tạo các nút lựa chọn đáp án
options = []
for i in range(4):
    btn = tk.Button(window, text="", font=("Arial", 14), width=30, command=lambda i=i: check_answer(i))
    btn.pack(pady=5)
    options.append(btn)

# Nút câu trước và câu tiếp theo
previous_button = tk.Button(window, text="Quay Lại", command=previous_question, font=("Arial", 14), bg="#2196F3", fg="white")
previous_button.pack(side=tk.LEFT, padx=20, pady=20)

next_button = tk.Button(window, text="Tiếp Theo", command=next_question, font=("Arial", 14), bg="#4CAF50", fg="white")
next_button.pack(side=tk.RIGHT, padx=20, pady=20)

# Nút thêm câu hỏi
add_button = tk.Button(window, text="Thêm Câu Hỏi", command=add_question, font=("Arial", 14), bg="#FF9800", fg="white")
add_button.pack(pady=10)

# Nút nhập câu hỏi từ file Excel
import_button = tk.Button(window, text="Nhập Câu Hỏi", command=import_questions, font=("Arial", 14), bg="#FF9800", fg="white")
import_button.pack(pady=10)

# Nhãn trạng thái
status_label = tk.Label(window, text="", font=("Arial", 12), bg="#f0f0f0")
status_label.pack(pady=10)

# Bảng hiển thị trạng thái các câu hỏi
question_table_frame = tk.Frame(window, bg="#f0f0f0")
question_table_frame.pack(pady=10)

question_buttons = []
for i in range(20):  # Giả sử có 20 câu hỏi tối đa
    btn = tk.Button(question_table_frame, text=str(i+1), width=4, height=2, bg="white", font=("Arial", 12))
    btn.grid(row=i//10, column=i%10, padx=5, pady=5)
    question_buttons.append(btn)

# Tải câu hỏi từ file
load_questions()

# Nếu có câu hỏi, hiển thị câu hỏi đầu tiên
if questions:
    load_question()
    update_status()
else:
    messagebox.showinfo("Không Có Câu Hỏi", "Không có câu hỏi nào. Vui lòng thêm câu hỏi để bắt đầu.")

# Khởi chạy ứng dụng
window.mainloop()
