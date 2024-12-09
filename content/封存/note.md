模型、模板、视图


---

```shell
(base) PS C:\Users\Administrator\Desktop\NewProject> cd django_project
(base) PS C:\Users\Administrator\Desktop\NewProject\django_project> python manage.py makemigrations enzysearcher
(base) PS C:\Users\Administrator\Desktop\NewProject\django_project> python manage.py migrate
(base) PS C:\Users\Administrator\Desktop\NewProject\django_project> python manage.py shell
In [1]: from enzysearcher.models import Enzyme
In [2]: a = Enzyme() # 注意这里要加括号
In [3]: a.enzyme_name = 'name'
In [6]: a.save() # 注意这里要加括号
In [7]: exit()
(base) PS C:\Users\Administrator\Desktop\NewProject\django_project> python manage.py createsuperuser  # 每次删除数据库后都要重新创建管理员
(base) PS C:\Users\Administrator\Desktop\NewProject\django_project> python manage.py runserver
```


```
text_pdf = 目标：从 {form_ref_0} 精确提取并呈现酶家族成员的生化数据，使用指定的 JSON 结构。强调提取过程中的准确性和细节。优先信息在术语定义中详细说明。  
  
术语定义：SPECIES: 生物的学名。仅使用双名法，不使用通用名称。  
  
ENZYME_COMMON_NAME: 酶的简短标识符，通常包含与属和种匹配的字母，后跟大写字母（有时是 ENZYME_FULL_NAME 的缩写）。  
  
ENZYME_FULL_NAME: 酶的长标识符，通常描述其功能。  
  
GENBANK: NCBI GenBank 核苷酸或蛋白质序列 ID，也称为“登录号”。作者可能会提到提交序列。  
  
UNIPROT_ID: UniProt（通用蛋白质资源）蛋白质序列 ID。  
  
ALT_ID: 任何其他不匹配 GENBANK 或 UNIPROT_ID 的序列标识符。  
  
SUBSTRATE: 酶作用的化学物质/分子。  
  
PRODUCT: 酶催化的反应产生的化学物质/分子。  
  
输出格式：  
  
字段名为以下的 JSON 行：SPECIES, ENZYME_COMMON_NAME, ENZYME_FULL_NAME, GENBANK, UNIPROT_ID, ALT_ID, SUBSTRATE, PRODUCT。对于不可用的数据使用 'NA'。  
  
指令：  
  
数据识别：阅读 {form_ref_1} 以识别关于酶介导反应的优先信息。  
  
数据提取：从 {form_ref_2} 中提取每个酶的数据，包括名称、底物、产物和序列 ID。  
  
全面包含：如果最初遗漏了序列 ID 信息，请再次审查文本。信息可能分散或与主要内容分开呈现，但论文中存在这些信息。  
  
JSON 行表示：将每个独特的酶-底物-产物相互作用（反应）表示为单独详细的 JSON 行。这种方法意味着单个酶催化多种反应时可能会有多个 JSON 行，每对独特的底物-产物组合都有自己的 JSON 行。  
  
格式化多个项：对于涉及多个底物或产物的反应，用分号（';'）分隔每个底物或产物。  
  
输出考虑：不要因空间限制而限制输出。确保呈现所有优先信息，而不提供解释或资格说明。如果呈现每个独特的酶-底物-产物相互作用需要多条消息，请按需发送额外的消息。您的输出应仅包含指定酶家族成员的数据。  
  
示例短语及术语：  
  
“来自 [SPECIES] 的酶 [ENZYME_COMMON_NAME] 催化 [SUBSTRATE] 转化为 [PRODUCT]。”\n“确认了 [ENZYME_COMMON_NAME] 对 [SUBSTRATE] 的活性，产生 [PRODUCT]。”\n“[SPECIES] 来源的 [ENZYME_COMMON_NAME] 的动力学研究表明对 [SUBSTRATE] 具有高特异性，导致 [PRODUCT] 形成。”\n“我们研究了 [ENZYME_COMMON_NAME] ([ENZYME_COMMON_NAME]) 是否可以使用 [SUBSTRATE]。”\n\n最终审查：\n\n在生成输出 JSON 行之前，请考虑您的输出和论文。填写或更正任何底物或产物信息。您的最终响应将仅包含每个独特酶-底物-产物相互作用（反应）的 JSON 对象。  
  
merge = 目标：将从科学文章 PDF 中通过不同过程生成的两个独立 JSON 对象列表中的生化数据合并为一个单一的综合 JSON 输出。合并后的输出应包含每个输入中的所有信息，避免冗余，确保酶家族成员的全面和准确表示。输出格式必须与输入格式相同，强调条目的精确合并。\n\n术语定义：\n\nSPECIES: 生物的学名。仅使用双名法，不使用通用名称。\nENZYME_COMMON_NAME: 酶的简短标识符，通常包含与属和种匹配的字母，后跟大写字母（有时是 ENZYME_FULL_NAME 的缩写）。\nENZYME_FULL_NAME: 酶的长标识符，通常描述其功能。\nGENBANK: NCBI GenBank 核苷酸或蛋白质序列 ID，也称为“登录号”。作者可能会提到提交序列。\nUNIPROT_ID: UniProt（通用蛋白质资源）蛋白质序列 ID。\nALT_ID: 任何其他不匹配 GENBANK 或 UNIPROT_ID 的序列标识符。\nSUBSTRATE: 酶作用的化学物质/分子。\nPRODUCT: 酶催化的反应产生的化学物质/分子。\n输出格式：\n字段名为以下的 JSON 行：SPECIES, ENZYME_COMMON_NAME, ENZYME_FULL_NAME, GENBANK, UNIPROT_ID, ALT_ID, SUBSTRATE, PRODUCT。对于不可用的数据使用 'NA'。\n\n指令：\n\n输入考虑：接受两个 JSON 对象列表作为输入，每个列表代表从科学文章中提取生化数据的不同过程的输出。这些对象包含酶数据字段：SPECIES, ENZYME_COMMON_NAME, ENZYME_FULL_NAME, GENBANK, UNIPROT_ID, ALT_ID, SUBSTRATE, PRODUCT。\n数据合并：将两个 JSON 列表输入中的数据合并为一个 JSON 列表输出。确保所有独特的酶-底物-产物相互作用都得以保留。如果两个输入包含相同的相互作用，将其合并为一个条目以避免冗余。\n全面包含：确保合并过程中没有数据丢失。两个输入 JSON 对象中存在的所有信息都应在合并输出中表示。\n输出格式：生成字段与输入相同的 JSON 行，用于合并输出：SPECIES, ENZYME_COMMON_NAME, ENZYME_FULL_NAME, GENBANK, UNIPROT_ID, ALT_ID, SUBSTRATE, PRODUCT。对于不可用的数据使用 'NA'。\n处理多个项：对于涉及多个底物或产物的条目，保持用分号（';'）分隔，如同原始格式。\n输出考虑：不要因空间限制而限制输出。确保呈现所有优先信息，而不提供解释或资格说明。如果呈现每个独特的酶-底物-产物相互作用需要多条消息，请按需发送额外的消息。\n\n示例合并情况：\n\n如果一个输入 JSON 对象列出了另一个未提及的酶的底物-产物对，将此对包含在最终输出中。\n对于跨输入匹配的酶条目，如果细节不同（例如，额外的 SUBSTRATE 或 PRODUCT 信息），将所有不同的信息编译为该酶的单个 JSON 行。\n\n最终审查：\n\n在最终确定合并的 JSON 输出之前，审查合并数据的完整性和准确性。纠正底物或产物信息中的任何差异或冗余，确保最终输出是科学文章中提取的生化数据的全面表示。最终响应将仅包含合并的 JSON 对象，每个独特的酶-底物-产物相互作用一个，不会丢失原始输出中的任何信息。  
```