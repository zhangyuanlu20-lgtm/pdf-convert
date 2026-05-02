import pdfplumber
import pandas as pd
import re

pdf_path = "Account_Statement.pdf"

rows = []
current = {}

with pdfplumber.open(pdf_path) as pdf:
    for page in pdf.pages:
        text = page.extract_text()
        lines = text.split("\n")

        for line in lines:
            line = line.strip()

            # 1️⃣ 匹配日期（新交易开始）
            if re.match(r"\d{2}/\d{2}/\d{4}", line):
                if current:
                    rows.append(current)
                    current = {}

                current["Date"] = line
                current["Description"] = ""

            # 2️⃣ 匹配金额行（关键）
            elif re.search(r"\d{1,3}(,\d{3})*\.\d{2}", line):
                nums = re.findall(r"-?\d{1,3}(?:,\d{3})*\.\d{2}", line)

                if len(nums) >= 3:
                    current["Deposit"] = nums[0]
                    current["Withdrawal"] = nums[1]
                    current["Balance"] = nums[2]

            # 3️⃣ 描述（拼接多行）
            else:
                if "Description" in current:
                    current["Description"] += " " + line

# 最后一条
if current:
    rows.append(current)

# 转DataFrame
df = pd.DataFrame(rows)
