dict1 = {"name": "현제", "old": 27, "job": "Developer"}
dict2 = {"address": "Seoul"}
dict3 = {**dict1, **dict2}
print(dict3)# {'name': '현제', 'old': 27, 'job': 'Developer', 'address': 'Seoul'}

dict4 = {"address": "homeless"}
dict5 = {**dict1, **dict2, **dict4}
print(dict5) # {'name': '현제', 'old': 27, 'job': 'Developer', 'address': 'homeless'}

dict6 = {**dict4, "status": "hungry"}
print(dict6) # {'address': 'homeless', 'status': 'hungry'}

from collections import Counter

# Example: 문자열의 각 문자 개수 세기
count = Counter("hello world")
print(count)  # Counter({'l': 3, 'o': 2, 'h': 1, 'e': 1, ' ': 1, 'w': 1, 'r': 1, 'd': 1})

# 잊을 법한 문법
dict6.update({"car": None, "computer": "Mac"}) # {'address': 'homeless', 'status': 'hungry', 'car': None, 'computer': 'Mac'}
print(dict6)
