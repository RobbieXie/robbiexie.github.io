---
title: manus-study
date: 2025-03-26 10:27:33
tags:

---

# Manus及相关技术栈

### manus

#### 官方示例

[manus.im](https://manus.im/)

**ubuntu@sanbox虚拟机**

**Terminal**: execute_command

![](./2025-03-26-14-20-28-image.png)

**File Editor**: create_file, edit_file

![](./2025-03-26-14-22-24-image.png)

**Browser**: browse_file, view_page(需要vllm), scroll_up, scroll_down

![](./2025-03-26-17-13-21-image.png)

**用户打断**，新增了几个简历 , 之前制定的 todo.md 内容会新增

![](./2025-03-26-17-14-54-image.png)

**用户要求** 将结果写入excel，agent编写代码实现

execute_command(pip instatll dependencies) + write_file(python script) + execute_command(python xxx.py)

![](./2025-03-26-17-28-45-image.png)

最终交付了excel xlsx文件

#### 其他案例replays

✅ [Comprehensive Tesla Stock Analysis and Investment Insights](https://manus.im/share/xFgpHb15vKqfRPWIs3JJPJ?replay=1)

❌ [Super Mario Game in Minecraft Style](https://manus.im/share/nrEydgtF8bLZFeDI8mPpUL?replay=1)

#### 攻击破解

<img title="" src="./2025-03-27-15-08-32-image.png" alt="" width="377">

prompt破解：[Manus tools and prompts · GitHub](https://gist.github.com/jlia0/db0a9695b3ca7609c9b1a08dcbf872c9) 

虚拟机文件破解：[manus-open(manus sandbox) · Github](https://github.com/whit3rabbit/manus-open)  [Replay: Check Files Under /opt/.manus Path - Manus](https://manus.im/share/lLR5uWIR5Im3k9FCktVu0k?replay=1)

相关资料：[分析](https://cn-sec.com/archives/3845527.html)

![](./2025-03-27-14-05-43-image.png)

### 相关开源项目

[Building Effective AI Agents \ Anthropic 对Agent的定义](https://www.anthropic.com/engineering/building-effective-agents)

<img title="" src="./2025-03-28-09-07-01-image.png" alt="" width="443">

#### OpenHands(OpenDevin)

[GitHub - All-Hands-AI/OpenHands: 🙌 OpenHands: Code Less, Make More](https://github.com/All-Hands-AI/OpenHands/tree/main)

![](./2025-03-27-11-34-31-image.png)

<img title="" src="./2025-03-27-14-10-23-image.png" alt="" width="324">

#### OpenManus

[👋 OpenManus 40k star](https://github.com/mannaandpoem/OpenManus)

google_search搜索引擎 + browser-use浏览器 + python exec代码执行 + subprocess bash执行 + 基础文件操作 

<img src="./2025-03-27-18-16-45-image.png" title="" alt="" width="210">

ManusAgent -> ToolCallAgent -> ReActAgent 

**测试prompt**：打开Bing搜索 搜索一下一招大模型的信息，多看几篇网页，总结一下重要信息

model: google/gemini-2.0-pro-exp-02-05

```json
{
  "messages": [
    {
      "role": "system",
      "content": "You are OpenManus, an all-capable AI assistant, aimed at solving any task presented by the user. You have various tools at your disposal that you can call upon to efficiently complete complex requests. Whether it's programming, information retrieval, file processing, or web browsing, you can handle it all."
    },
    {
      "role": "user",
      "content": "打开Bing搜索 搜索一下一招大模型的信息，多看几篇网页，总结一下重要信息"
    },
    {
      "role": "user",
      "content": "You can interact with the computer using PythonExecute, save important content and information files through FileSaver, open browsers with BrowserUseTool, and retrieve information using BingSearch.\n\nPythonExecute: Execute Python code to interact with the computer system, data processing, automation tasks, etc.\n\nFileSaver: Save files locally, such as txt, py, html, etc.\n\nBrowserUseTool: Open, browse, and use web browsers.If you open a local HTML file, you must provide the absolute path to the file.\n\nBingSearch: Perform web information retrieval, and return ref links, you must use BrowserUseTool to fetch information from these links.\n\nBased on user needs, proactively select the most appropriate tool or combination of tools. For complex tasks, you can break down the problem and use different tools step by step to solve it. After using each tool, clearly explain the execution results and suggest the next steps.\n"
    },
    {
      "role": "assistant",
      "content": "Okay, I will start by searching for information about \"一招大模型\" using Bing Search. Then, I will browse the search results and summarize the important information.\n\n\n",
      "tool_calls": [
        {
          "id": "tool_0_bing_search",
          "type": "function",
          "function": {
            "name": "bing_search",
            "arguments": "{\"num_results\":5,\"query\":\"一招大模型\"}"
          }
        }
      ]
    },
    {
      "role": "tool",
      "content": "Observed output of cmd `bing_search` executed:\n['https://cj.sina.com.cn/articles/view/5816954579/15ab7aed301901e374', 'https://www.modelscope.cn/models/CMB_AILab/YiZhao-12B-Chat/summary', 'https://finance.sina.cn/2025-01-03/detail-inecsvux6431783.d.html', 'https://finance.sina.com.cn/tob/2024-12-11/doc-inczafmv0853031.shtml', 'https://baijiahao.baidu.com/s?id=1818122776352809538']",
      "name": "bing_search",
      "tool_call_id": "tool_0_bing_search"
    },
    {
      "role": "user",
      "content": "You can interact with the computer using PythonExecute, save important content and information files through FileSaver, open browsers with BrowserUseTool, and retrieve information using BingSearch.\n\nPythonExecute: Execute Python code to interact with the computer system, data processing, automation tasks, etc.\n\nFileSaver: Save files locally, such as txt, py, html, etc.\n\nBrowserUseTool: Open, browse, and use web browsers.If you open a local HTML file, you must provide the absolute path to the file.\n\nBingSearch: Perform web information retrieval, and return ref links, you must use BrowserUseTool to fetch information from these links.\n\nBased on user needs, proactively select the most appropriate tool or combination of tools. For complex tasks, you can break down the problem and use different tools step by step to solve it. After using each tool, clearly explain the execution results and suggest the next steps.\n"
    },
    {
      "role": "assistant",
      "content": "Now I will browse the search results to gather information. I will start with the first link.\n",
      "tool_calls": [
        {
          "id": "tool_0_browser_use",
          "type": "function",
          "function": {
            "name": "browser_use",
            "arguments": "{\"url\":\"https://cj.sina.com.cn/articles/view/5816954579/15ab7aed301901e374\",\"action\":\"navigate\"}"
          }
        }
      ]
    },
    {
      "role": "tool",
      "content": "Observed output of cmd `browser_use` executed:\nNavigated to https://cj.sina.com.cn/articles/view/5816954579/15ab7aed301901e374",
      "name": "browser_use",
      "tool_call_id": "tool_0_browser_use"
    },
    {
      "role": "user",
      "content": "You can interact with the computer using PythonExecute, save important content and information files through FileSaver, open browsers with BrowserUseTool, and retrieve information using BingSearch.\n\nPythonExecute: Execute Python code to interact with the computer system, data processing, automation tasks, etc.\n\nFileSaver: Save files locally, such as txt, py, html, etc.\n\nBrowserUseTool: Open, browse, and use web browsers.If you open a local HTML file, you must provide the absolute path to the file.\n\nBingSearch: Perform web information retrieval, and return ref links, you must use BrowserUseTool to fetch information from these links.\n\nBased on user needs, proactively select the most appropriate tool or combination of tools. For complex tasks, you can break down the problem and use different tools step by step to solve it. After using each tool, clearly explain the execution results and suggest the next steps.\n"
    },
    {
      "role": "assistant",
      "content": "Now I will get the text content of the page.\n",
      "tool_calls": [
        {
          "id": "tool_0_browser_use",
          "type": "function",
          "function": {
            "name": "browser_use",
            "arguments": "{\"action\":\"get_text\"}"
          }
        }
      ]
    },
    {
      "role": "tool",
      "content": "Observed output of cmd `browser_use` executed:\n注册\n登录\n新浪财经APP\n宏观经济\nA股\n港股\n美股\n基金\n理财\n黄金\n期货\n能源\n互联网金融\n房地产\n更多\n招行有“一招（YiZhao）”\n2024年12月12日 08:01 支付之家网\n语音播报\n缩小字体\n放大字体\n微博\n微信\n分享\n0\n\n在快速发展的数字化浪潮中，银行业正面临着一次前所未有的变革。\n\n近日，招商银行申请注册了“一招(YiZhao)大模型”和“招商银行一招(YiZhao)大模型”商标，这一举动不仅让人对招行的科技布局充满了期待，也透露出银行在人工智能（AI）领域的深远布局。这一商标涉及多个国际分类领域，包括广告销售、网站服务、科学仪器等，预示着招行可能会推出更多基于AI大模型技术的创新产品或服务。\n\n招行作为金融行业的科技创新先锋，一直以来都在数字化转型方面走在行业的前列。从推出“一卡通”到开发智能客服“招小宝”，再到利用区块链技术打造金融服务平台，招行始终致力于提升客户体验、优化服务效率。\n\n如今，招行提出的“一招”大模型，不仅是一次技术创新，更是对银行业服务模式的一次大胆创新，它标志着银行行业将与人工智能深度融合，进入一个全新的智能化服务时代。\n\n大模型与金融科技的深度融合\n\n以ChatGPT为代表的大模型技术，展示了强大的自然语言处理与生成能力，其应用已拓展至教育、医疗、法律等多个领域。然而，银行业作为一个数据密集且高度专业化的行业，对大模型提出了更高的要求：不仅需要对行业术语、法律法规以及复杂的客户需求进行精确理解，还需要在服务个性化、运营自动化和风险控制等方面提供专业的解决方案。\n\n1. 从通用模型到行业模型\n\n虽然像ChatGPT这样的通用模型可以处理广泛的问题，但其在金融场景中的表现需要进一步优化。例如，银行客服需要处理精准的账户查询，投资顾问需要结合客户的财务数据和市场走势提供个性化建议。这些需求要求大模型具备更强的专业化和场景化能力。\n\n2. 金融行业的智慧化转型\n\n银行业的数字化转型已迈入深水区，未来的竞争核心在于智慧化服务。大模型不仅可以作为客户的智能助手，为其提供洞察性的理财建议；也可以作为员工的数字化伙伴，帮助其提升工作效率、优化决策过程。\n\n招商银行信息技术部副总经理俞吴杰透露，针对AI与大语言模型，目前招行的重点在于智算平台的建设。“原来的服务器集群叫通算平台，和大模型相关的叫智算平台。智算平台既包含招行的训练平台，也包括推理平台。”他提到，招行探索大模型的目标不是建立通用模型，而是在金融行业建设垂直领域的大模型。\n\n招行的创新不仅局限于大模型的应用，还深入探索了低代码平台与大模型的结合。低代码开发可以大幅降低开发门槛，让业务用户也能参与到系统开发中来。而大模型的引入，使得低代码开发过程变得更加高效和智能。\n\n招商银行自2019年启动了企业级低代码开发平台的建设，五年来，低代码平台已成为业务系统开发的重要基础，支持零代码和低代码两种开发模式。尤其在2023年，招行进一步探索了低代码与大模型的融合，提升了开发者的体验，推动了银行业务的科技平民化。\n\n大模型驱动的银行业智慧化转型\n\n银行业的竞争日益激烈，未来的核心竞争力将在于智慧化服务。大模型技术不仅仅是在后台支持银行的智能化运作，更是在客户交互、金融服务等方面发挥着重要作用。\n\n在银行业中，大模型技术的落地应用正在重塑行业生态，其核心场景包括：\n\n1. 智能客服：更主动的客户体验\n\n大模型支持从被动响应向主动交互转变。传统客服主要解答客户的固定问题，而大模型赋能下的智能客服可以实时分析客户意图，提供个性化建议。例如，在客户咨询投资时，客服不仅能快速匹配合适的理财产品，还能生成详细的市场分析和风险评估。\n\n2. 风险控制：实时监测与动态优化\n\n金融风险管理是银行的核心业务之一。大模型通过对海量交易数据的实时分析，能够迅速识别异常行为，提高欺诈检测的精度。同时，它还可以结合外部政策、经济数据，动态调整风控模型，为银行提供更可靠的风险管理支持。\n\n3. 个性化投顾：精准洞察客户需求\n\n传统的理财服务往往以人工顾问为主，但其覆盖面和效率有限。大模型可以分析客户的财务状况、风险偏好以及历史行为数据，生成个性化的投资建议，从而覆盖更多长尾客户，提升服务普惠性。\n\n4. 内部流程优化：高效协同与智能辅助\n\n银行内部流程复杂，例如贷款审批、合同审阅等，通常需要耗费大量时间。大模型可以实现这些流程的自动化处理，不仅节约人力成本，还能大幅提高效率。例如，它可以在合同审阅中快速标记风险条款，并给出修改建议。\n\n5. 合规性与透明度：监管与服务的双赢\n\n金融行业的高度合规性要求对技术提出了更高标准。大模型可以实时监控业务流程，确保操作符合监管要求。同时，它还能生成易于理解的报告，帮助客户更清楚地了解自身的金融行为和合规情况。\n\n招行“一招”的潜力与战略意义\n\n此次“一招”大模型商标的申请，展示了招行在数字化与智能化转型上的战略决心。结合其多年积累的技术优势和场景深耕，“一招”有望成为银行业大模型应用的标杆。\n\n1. 数据、技术与场景的深度融合\n\n招行拥有丰富的客户数据和复杂的业务场景，这为训练和优化大模型提供了强大的基础。通过将技术与实际业务深度结合，招行能够开发出更贴近客户需求、更具竞争力的智能服务。\n\n2. 从银行服务到金融生态的扩展\n\n“一招”大模型的潜力不仅限于银行内部业务，还可能扩展到更广阔的金融生态。例如，通过开放接口，与中小企业和金融科技公司合作，提供智能风控、自动化运营等服务，助力整个行业的数字化升级。\n\n3. 智能化驱动的服务升级\n\n从“一卡通”到“掌上生活”APP，招行一直致力于提升客户体验。“一招”大模型的推出，将进一步推动银行服务从标准化向个性化、从单向服务向双向交互转型，为客户带来更加高效、贴心的金融服务。\n\n挑战与机遇：技术落地的关键问题\n\n招行在低代码与大模型领域的创新实践，标志着其数字化转型进程中的重要一步。然而，尽管大模型在银行业的应用前景广阔，但其实际落地仍面临一些挑战：\n\n数据安全与隐私保护\n\n银行掌握着大量敏感数据，大模型的开发和使用需要确保信息的安全性与合规性，避免客户隐私泄露。\n\n模型的可解释性与透明性\n\n在金融场景中，决策结果必须具备高可解释性，既要让客户信服，也要满足监管要求。这对大模型的技术设计提出了更高的要求。\n\n技术迭代与生态协同\n\n大模型技术发展迅速，银行需要保持技术敏锐度，同时建立与外部技术生态的良好协同，以实现持续的创新与突破。\n\n尽管面临挑战，招行凭借其在技术上的深厚积累，已经在人工智能、大模型和低代码开发方面积累了宝贵经验，未来将在智能化服务、金融生态的建设上继续推动行业的革新。\n\n“一招”大模型的商标申请，不仅标志着招行在人工智能领域迈出了重要一步，也揭示了未来银行业智慧化服务的趋势：以技术为驱动，从底层重塑客户体验和运营模式。\n\n在这个技术与金融深度融合的时代，招行以“一招”出击，向行业展示了金融科技创新的无限可能。未来，我们期待看到招行将这一“妙招”变为实招，助力客户与行业在智能时代中共创辉煌。\n\n注：本文为支付之家网投稿，作者系业内从业人员，文中观点仅代表作者个人观点\n\n0条评论|0人参与\n网友评论\n登录|注册\n发布\n相关新闻\n加载中 \n头条号入驻\n支付之家网\n支付之家网(ZFZJ.CN)官方头条号！\n大阪严禁老人在ATM机前打电话丨试点向移动支付平台开放信用监管数据\n今日截止，国资支付牌照挂牌转让！\n央视曝光，2家支付机构被“另案处理”！\n财经自媒体联盟\n今日推荐\n优秀作者\n看点月榜\nCEO下了飞机就来谈合作！为何外企纷纷看多中国？｜宅男财经\n\n中新经纬\n\n实测完DeepSeek发布的新模型，我觉得AI编程的全民普惠时刻\n\n数字生命卡兹克\n\n疯狂小杨哥要回归了？三只羊公司累计赔付近三千万 具备恢复经营条件\n\n雷达财经\n\n阿里蔡崇信：民企座谈会被低估了，人工智能市场10万亿美金\n\n盈利靠“节衣缩食”，虎牙的日子越来越难过了\n\n互联网分析师于斌\n\n新浪财经头条意见反馈留言板\n\n4000520066 欢迎批评指正\n\n关于头条 | 如何入驻 | 发稿平台 | 奖励机制 版权声明 | 用户协议 | 帮助中心\nCopyright © 1996-2019 SINA Corporation\nAll Rights Reserved 新浪公司 版权所有\n新浪首页\n语音播报\n相关新闻\n返回顶部",
      "name": "browser_use",
      "tool_call_id": "tool_0_browser_use"
    },
    {
      "role": "user",
      "content": "You can interact with the computer using PythonExecute, save important content and information files through FileSaver, open browsers with BrowserUseTool, and retrieve information using BingSearch.\n\nPythonExecute: Execute Python code to interact with the computer system, data processing, automation tasks, etc.\n\nFileSaver: Save files locally, such as txt, py, html, etc.\n\nBrowserUseTool: Open, browse, and use web browsers.If you open a local HTML file, you must provide the absolute path to the file.\n\nBingSearch: Perform web information retrieval, and return ref links, you must use BrowserUseTool to fetch information from these links.\n\nBased on user needs, proactively select the most appropriate tool or combination of tools. For complex tasks, you can break down the problem and use different tools step by step to solve it. After using each tool, clearly explain the execution results and suggest the next steps.\n"
    },
    {
      "role": "assistant",
      "content": "The text content from the first search result is now available. It appears to be an article about \"一招(YiZhao)大模型\" from a financial perspective, specifically focusing on the application by China Merchants Bank (招商银行).\n\nHere's a summary of the key information:\n\n*   **招商银行 (China Merchants Bank) is developing an AI model called \"一招(YiZhao)\"**: They have applied for trademarks related to this model.\n*   **Focus on Financial Applications**: The model is intended for use in various financial services, including customer service, risk control, and personalized investment advice.\n*   **Integration with Low-Code Development**:  招商银行 is also integrating the model with low-code development platforms to improve efficiency.\n*   **Benefits**: The article highlights potential benefits such as improved customer experience, operational efficiency, and risk management.\n*   **Challenges**: The article also mentions challenges like data security, model explainability, and the need for continuous technological advancements.\n\nNow, I will proceed to the second search result to gather more information.\n",
      "tool_calls": [
        {
          "id": "tool_0_browser_use",
          "type": "function",
          "function": {
            "name": "browser_use",
            "arguments": "{\"url\":\"https://www.modelscope.cn/models/CMB_AILab/YiZhao-12B-Chat/summary\",\"action\":\"navigate\"}"
          }
        }
      ]
    },
    {
      "role": "tool",
      "content": "Observed output of cmd `browser_use` executed:\nError: Browser action 'navigate' failed: Page.goto: Timeout 30000ms exceeded.\nCall log:\n  - navigating to \"https://www.modelscope.cn/models/CMB_AILab/YiZhao-12B-Chat/summary\", waiting until \"load\"\n",
      "name": "browser_use",
      "tool_call_id": "tool_0_browser_use"
    },
    {
      "role": "user",
      "content": "You can interact with the computer using PythonExecute, save important content and information files through FileSaver, open browsers with BrowserUseTool, and retrieve information using BingSearch.\n\nPythonExecute: Execute Python code to interact with the computer system, data processing, automation tasks, etc.\n\nFileSaver: Save files locally, such as txt, py, html, etc.\n\nBrowserUseTool: Open, browse, and use web browsers.If you open a local HTML file, you must provide the absolute path to the file.\n\nBingSearch: Perform web information retrieval, and return ref links, you must use BrowserUseTool to fetch information from these links.\n\nBased on user needs, proactively select the most appropriate tool or combination of tools. For complex tasks, you can break down the problem and use different tools step by step to solve it. After using each tool, clearly explain the execution results and suggest the next steps.\n"
    },
    {
      "role": "assistant",
      "content": "The attempt to navigate to the second URL timed out. I will try again.\n",
      "tool_calls": [
        {
          "id": "tool_0_browser_use",
          "type": "function",
          "function": {
            "name": "browser_use",
            "arguments": "{\"url\":\"https://www.modelscope.cn/models/CMB_AILab/YiZhao-12B-Chat/summary\",\"action\":\"navigate\"}"
          }
        }
      ]
    },
    {
      "role": "tool",
      "content": "Observed output of cmd `browser_use` executed:\nNavigated to https://www.modelscope.cn/models/CMB_AILab/YiZhao-12B-Chat/summary",
      "name": "browser_use",
      "tool_call_id": "tool_0_browser_use"
    },
    {
      "role": "user",
      "content": "You can interact with the computer using PythonExecute, save important content and information files through FileSaver, open browsers with BrowserUseTool, and retrieve information using BingSearch.\n\nPythonExecute: Execute Python code to interact with the computer system, data processing, automation tasks, etc.\n\nFileSaver: Save files locally, such as txt, py, html, etc.\n\nBrowserUseTool: Open, browse, and use web browsers.If you open a local HTML file, you must provide the absolute path to the file.\n\nBingSearch: Perform web information retrieval, and return ref links, you must use BrowserUseTool to fetch information from these links.\n\nBased on user needs, proactively select the most appropriate tool or combination of tools. For complex tasks, you can break down the problem and use different tools step by step to solve it. After using each tool, clearly explain the execution results and suggest the next steps.\n"
    },
    {
      "role": "assistant",
      "content": "Now I will get the text content of the page.\n\n",
      "tool_calls": [
        {
          "id": "tool_0_browser_use",
          "type": "function",
          "function": {
            "name": "browser_use",
            "arguments": "{\"action\":\"get_text\"}"
          }
        }
      ]
    },
    {
      "role": "tool",
      "content": "Observed output of cmd `browser_use` executed:\n首页\n模型库\n数据集\n创空间\nAIGC专区\n文档中心\n社区\nGitHub\n登录 / 注册\n一招-12B-Chat\nCMB_AILab/YiZhao-12B-Chat\nYiZhao\n@招商银行人工智能实验室提供64下载2025-01-14更新\nNotebook快速开发\n喜欢\n加入合集\n模型介绍\n模型文件\n交流反馈3\n一招（YiZhao)使用指南\n\n大模型金融能力榜单 • 模型文件 • 开源数据\n\n目录\n模型介绍\n效果评测\n快速开始\n声明与协议\n最新动态\n2024年12月10日 开源2TB高质量多模态的大模型训练数据集\n模型介绍\n\n一招（YiZhao)是招商银行联合华为、智谱AI，在全国产的设备与训练框架下训练形成的金融领域大语言模型系列。我们首先在大规模高质量数据上进行预训练，再通过高质量金融数据进行后期微调（SFT），最后通过直接偏好优化（DPO）进一步优化模型，最终训练出了YiZhao-12B-Chat。YiZhao-12B-Chat具备自然语言理解、文本生成、舆情事件抽取、工具使用、对话互动等多种功能。YiZhao-12B-Chat是一个专为金融领域设计的 120亿参数大型语言模型，支持32K上下文长度。 主要特点包括：\n\n对海量通用语料与金融语料进行了多维度数据清洗与筛选，保证训练数据的数量和质量。\n\nYiZhao-12B-Chat是基于GLM（General Language Model）架构的中英双语对话模型，具有120亿参数，对问答和对话场景进行了优化。\n\nYiZhao-12B-Chat完全基于国产算力和国产深度学习框架MindSpore进行训练，算力和算法框架更自主可控。\n\n这款模型在云端私有化部署后，可以为企业和个人提供高效、灵活的智能对话解决方案。\n\n下载链接：\n\n社区\t下载地址\n魔搭社区（mindspore）\thttps://www.modelscope.cn/models/CMB_AILab/YiZhao-12B-Chat/\n魔搭社区（safetensor）\thttps://modelscope.cn/models/CMB_AILab/YiZhao-12B-Chat-HF\n魔乐社区 （mindspore）\thttps://modelers.cn/models/CMB_AI_Lab/YiZhao-12B-Chat/\n码云\thttps://gitee.com/mindspore/mindformers/tree/dev/research/yizhao\n效果评测\n\n效果评测包括通用能力评测和金融领域评测。模型在保持通用能力的基础上进一步提升了金融领域能力。\n\n1. 通用评测\n\n在通用领域评测中，我们选择当下主流的几类客观评测基准，见下表：\n\n能力\t任务\t描述\n逻辑推理\tARC Challenge\tARC问题需要多种类型的知识与推理，包括定义、基本事实和属性、结构、过程与因果、目的论、代数、实验、空间/运动学、类比等，ARC问题集包含7787个自然科学问题\n中文知识\tCMMLU\t67个学科中文考试题\n数学能力\tGSM8K\t8.5k个样例，数学推理能力\n通用知识\tMMLU\tMMLU 是一个涵盖STEM、人文学科、社会科学等57个学科领域(例如：数学、法律、伦理等)的评测基准，旨在考察大语言模型将知识运用于问题解决的能力\n指令遵从\tIFEval\t确定了25种可验证的指令类型，500个包含一个或多个可验证指令的提示（prompts）\n测试结果如下：\n模型\t逻辑推理\t中文知识\t数学能力\t通用知识\t指令遵从\n\tarc_challenge\tcmmlu\tgsm8k\tmmlu\tifeval\n一招-12B-Chat\t0.9331\t0.7158\t0.8993\t0.7192\t0.5432\n小结：\n\nYiZhao-12B在通用评测集方面均有出色表现。\n\nYiZhao-12B的预训练方案并没有牺牲过多模型通用能力，而且增量训练数据中的中文金融数据，也一定程度地增强了模型的逻辑推理、中文、数学等能力。\n\n2. 金融评测\n\n金融评测主要包括以下三个测试：\n\n任务\t描述\nCCKS-IJCKG：DFMEB\t为了推动LLM在数字金融领域的发展，并解决实际金融业务问题。招商银行联合中科院自动化所、科大讯飞股份有限公司，结合实际生产场景，推出数字金融领域评测基准（Digital Finance Model Evaluation Benchmark，DFMEB）。该评测基准包含六大场景（知识问答、文本理解、内容生成、逻辑推理、安全合规、AI智能体），涵盖69种金融任务，有利于帮助开源社区和业界快速评测公开或者自研LLM。\nCFBenchmark-OpenFinData\t“书生•济世”中文金融评测基准（CFBenchmark）基础版本由CFBenchmark-Basic（全部为主观题，不参与测评）和CFBenchmark-OpenFinData两部分数据组成。OpenFinData是由东方财富与上海人工智能实验室联合发布的开源金融评测数据集。该数据集代表了最真实的产业场景需求，是目前场景最全、专业性最深的金融评测数据集。它基于东方财富实际金融业务的多样化丰富场景，旨在为金融科技领域的研究者和开发者提供一个高质量的数据资源。\nFinancelQ\tFinanceIQ是一个专业的金融领域评测集，其涵盖了10个金融大类及36个金融小类，总计7173个单项选择题，某种程度上可客观反应模型的金融能力。\n测试结果如下：\n模型\tCCKS-IJCKG：DFMEB\tCFBenchmark-OpenFinData\tFinancelQ\n一招-12B-Chat\t0.8218\t0.8798\t0.6867\n小结：\n\nYiZhao-12B-Chat在金融测评方面表现优异。\n\nYiZhao-12B-Chat有着较强的专业知识能力，在金融分析、金融考核、金融安全合规、风险检查等多个专业领域维度有着出色的表现。\n\n快速开始\n\nYiZhao-12B-Chat模型已发布在魔搭社区、魔乐社区，请根据上述链接进行下载。下方为模型的推理部署使用方法。\n\n依赖软件\n\n下表列出了运行一招推理所需的环境和第三方依赖。\n\n软件名称\t版本\t作用\nMindSpore\t2.4.0\t深度学习框架\nMindSpore Transformers\t1.3.0\t大模型全流程开发套件\nCANN\t8.0.RC3.beta1\t昇腾异构计算架构\nsympy\t1.13.3\t三方依赖包\nPython\t3.10\tMindSpore的使用依赖Python环境\nGCC\t7.3.0\t用于编译MindSpore的C++编译器\n\n注：MindSpore官网介绍：https://www.mindspore.cn/\n\n安装指导\n1. 安装CANN\n\n下载CANN社区版，选择8.0.RC3.beta1版本，按顺序执行：\n\n./Ascend-cann-toolkit_8.0.RC3_linux-aarch64.run --install --install-for-all --install-path=/usr/local/Ascend/\n./Ascend-cann-kernels-910b_8.0.RC3_linux-aarch64.run --install --install-path=/usr/local/Ascend/\n\n\n安装完毕后执行：\n\npip uninstall te topi hccl -y\npip install sympy\npip install /usr/local/Ascend/ascend-toolkit/latest/lib64/te-*-py3-none-any.whl\npip install /usr/local/Ascend/ascend-toolkit/latest/lib64/hccl-*-py3-none-any.whl\n\n2. 安装MindSpore\npip install https://ms-release.obs.cn-north-4.myhuaweicloud.com/2.4.0/MindSpore/unified/aarch64/mindspore-2.4.0-cp310-cp310-linux_aarch64.whl --trusted-host ms-release.obs.cn-north-4.myhuaweicloud.com -i https://pypi.tuna.tsinghua.edu.cn/simple\n\n\n验证MindSpore与CANN是否安装成功：\n\nsource /usr/local/Ascend/ascend-toolkit/set_env.sh # 每次运行时都需要设置此环境变量\npython -c \"import mindspore;mindspore.set_context(device_target='Ascend');mindspore.run_check()\"\n\n3. 安装MindSpore Transformers\npip install mindformers==1.3.0\n\n4. 安装openmind-hub\npip install openmind-hub  #如需自动下载，需要安装额外依赖openmind-hub\n\n代码示例\n\n此处提供一个代码示例，展示如何加载tokenizer和模型，并如何生成内容：\n\n\nimport os\nimport time\nimport mindspore as ms\nfrom mindformers.pipeline import pipeline\n\nos.environ[\"RUN_MODE\"] = \"predict\"\nos.environ[\"MS_INTERNAL_DISABLE_CUSTOM_KERNEL_LIST\"] = \"PagedAttention\"\nms.set_context(device_id=0, mode=0, device_target='Ascend', jit_config={'jit_level': 'O0', 'infer_boost': 'on'})\n\npipeline_task = pipeline(task=\"text_generation\",\n                         model=\"CMB_AILab/YiZhao-12B-Chat\",  # model_path\n                         framework='ms',\n                         model_kwargs={\"use_past\": True},\n                         trust_remote_code=True\n                         )\nwhile True:\n    input_ = input(\"模型输入: \")\n    template = f\"<|user|>{input_}<|assistant|>\"\n\n    pipeline_result = pipeline_task(template, do_sample=False, seed=int(time.time()))\n    print(pipeline_result[0][\"text_generation_text\"][0].replace(f\"{input_}\", \"\", 1))\n\n声明与协议\n声明\n\n我们在此声明，不要使用一招（YiZhao）模型及其衍生模型进行任何危害国家社会安全或违法的活动。同时，我们也要求使用者不要将一招（YiZhao）用于未经必要安全审查和备案的互联网服务。我们希望所有使用者遵守上述原则，确保科技发展在合法合规的环境下进行。\n\n我们已经尽我们所能，来确保模型训练过程中使用的数据及模型的合规性。然而，尽管我们已经做出了巨大的努力，但由于模型和数据的复杂性，仍有可能存在一些无法预见的问题。使用者应结合使用场景、使用目的、涉及数据敏感性、预期运行环境，进行必要的测试，了解了大模型局限性，评估潜在风险及使用大模型的合法性及适当性，采取必要的缓释措施，并履行必要的安全及合规义务。因使用一招（YiZhao）开源模型而导致的任何问题，包括但不限于数据安全问题、公共舆论风险，或模型被误导、滥用、传播或不当利用所带来的任何风险和问题，我们将不承担任何责任。\n\n协议\n\n社区使用一招（YiZhao）模型需要遵循《“一招（YiZhao）”许可协议》。\n\n如果使用方为非自然人实体，您需要通过以下联系邮箱ailab@cmbchina.com，提交《“一招（YiZhao）”许可协议》要求的申请材料。审核通过后，将特此授予您一个非排他性、全球性、不可转让、不可再许可、可撤销和免版税的有限许可，但仅限于被许可方内部的使用、复制、分发以及修改，并且应当遵守《“一招（YiZhao）”许可协议》。\n\n作者：\n招商银行人工智能实验室\n  \n模型 2\n数据集 0\n创空间 0\n收录本模型的合集（ 1 ）\nAIOps\n2025.03.20\n3\n0\n\n分享\n\n© 2022-2025 ModelScope.cn 版权所有",
      "name": "browser_use",
      "tool_call_id": "tool_0_browser_use"
    },
    {
      "role": "user",
      "content": "You can interact with the computer using PythonExecute, save important content and information files through FileSaver, open browsers with BrowserUseTool, and retrieve information using BingSearch.\n\nPythonExecute: Execute Python code to interact with the computer system, data processing, automation tasks, etc.\n\nFileSaver: Save files locally, such as txt, py, html, etc.\n\nBrowserUseTool: Open, browse, and use web browsers.If you open a local HTML file, you must provide the absolute path to the file.\n\nBingSearch: Perform web information retrieval, and return ref links, you must use BrowserUseTool to fetch information from these links.\n\nBased on user needs, proactively select the most appropriate tool or combination of tools. For complex tasks, you can break down the problem and use different tools step by step to solve it. After using each tool, clearly explain the execution results and suggest the next steps.\n"
    }
  ],
  "model": "google/gemini-2.0-flash-lite-preview-02-05:free",
  "max_tokens": 128000,
  "temperature": 0,
  "tool_choice": "auto",
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "python_execute",
        "description": "Executes Python code string. Note: Only print outputs are visible, function return values are not captured. Use print statements to see results.",
        "parameters": {
          "type": "object",
          "properties": {
            "code": {
              "type": "string",
              "description": "The Python code to execute."
            }
          },
          "required": [
            "code"
          ]
        }
      }
    },
    {
      "type": "function",
      "function": {
        "name": "bing_search",
        "description": "执行必应搜索并返回相关链接列表。\n当需要获取互联网信息或英文内容时建议使用此工具。\n工具返回与搜索查询匹配的URL列表。后续可以通过browser_use工具去进行访问URL",
        "parameters": {
          "type": "object",
          "properties": {
            "query": {
              "type": "string",
              "description": "(必填) 提交给必应的搜索关键词"
            },
            "num_results": {
              "type": "integer",
              "description": "(可选) 返回的搜索结果数量，默认5",
              "default": 5
            }
          },
          "required": [
            "query"
          ]
        }
      }
    }
  ]
}
```

#### LangManus

[ 🦜🤖 LangManus 5.2k star](https://github.com/langmanus/langmanus/tree/main)

基于langchain生态multiAgents实现类似功能，tavily搜索引擎 + jina爬虫 + browser-use浏览器 + pythonREPL代码执行 + subprocess bash执行 + 基础文件操作

<img title="" src="./2025-03-27-14-29-59-image.png" alt="" width="504" data-align="left">

系统由以下智能体协同工作：

1. **协调员（Coordinator）**：工作流程的入口点，处理初始交互并路由任务
2. **规划员（Planner）**：分析任务并制定执行策略
3. **主管（Supervisor）**：监督和管理其他智能体的执行
4. **研究员（Researcher）**：收集和分析信息
5. **程序员（Coder）**：负责代码生成和修改
6. **浏览器（Browser）**：执行网页浏览和信息检索 
7. **汇报员（Reporter）**：生成工作流结果的报告和总结

实现这些agents一共只用到了以下tools

<img title="" src="./2025-03-27-14-51-50-image.png" alt="" width="223" data-align="left">

由supervisor控制loop和结束, prompt如下

```markdown
You are a supervisor coordinating a team of specialized workers to complete tasks. Your team consists of: [{{ TEAM_MEMBERS|join(", ") }}].

For each user request, you will:

Analyze the request and determine which worker is best suited to handle it next
Respond with ONLY a JSON object in the format: {"next": "worker_name"}
Review their response and either:
Choose the next worker if more work is needed (e.g., {"next": "researcher"})
Respond with {"next": "FINISH"} when the task is complete
Always respond with a valid JSON object containing only the 'next' key and a single value: either a worker's name or 'FINISH'.

Team Members
{% for agent in TEAM_MEMBERS %}

{{agent}}: {{ TEAM_MEMBER_CONFIGRATIONS[agent]["desc_for_llm"] }} {% endfor %}
```

### 开源基础组件

#### Browser-use

[GitHub - browser-use/browser-use 🤖  49K star](https://github.com/browser-use/browser-use)

🌐 Browser-use is the easiest way to connect your AI agents with the browser. 它集成了开源浏览器自动化工具playwright来实现api控制浏览器行为，并内置了一个Agent，通过react让 llm loop执行直到完成用户指令。基本用法如下

```python
from langchain_openai import ChatOpenAI
from browser_use import Agent
import asyncio
from dotenv import load_dotenv
load_dotenv()

async def main():
    agent = Agent(
        task="Compare the price of gpt-4o and DeepSeek-V3",
        llm=ChatOpenAI(model="gpt-4o"),
        use_vision=True,  
    )
    await agent.run()

asyncio.run(main())
```

内置的浏览器tools：

```
go_to_url
go_back
wait
search_google
click_element
input_text
switch_tab
open_tab
extract_content
scroll_down
scroll_up
send_keys
scroll_to_text
get_dropdown_options
select_dropdown_option
done
```

LLM实际调用日志：react过程

**测试prompt**：打开Bing搜索 搜索一下一招大模型的信息，多看几篇网页，总结一下重要信息

model: groq/deepseek-r1-distill-qwen-32b

```json
{
  "messages": [
    {
      "content": "You are an AI agent designed to automate browser tasks. Your goal is to accomplish the ultimate task following the rules.\n\n# Input Format\nTask\nPrevious steps\nCurrent URL\nOpen Tabs\nInteractive Elements\n[index]<type>text</type>\n- index: Numeric identifier for interaction\n- type: HTML element type (button, input, etc.)\n- text: Element description\nExample:\n[33]<button>Submit Form</button>\n\n- Only elements with numeric indexes in [] are interactive\n- elements without [] provide only context\n\n# Response Rules\n1. RESPONSE FORMAT: You must ALWAYS respond with valid JSON in this exact format:\n{\"current_state\": {\"evaluation_previous_goal\": \"Success|Failed|Unknown - Analyze the current elements and the image to check if the previous goals/actions are successful like intended by the task. Mention if something unexpected happened. Shortly state why/why not\",\n\"memory\": \"Description of what has been done and what you need to remember. Be very specific. Count here ALWAYS how many times you have done something and how many remain. E.g. 0 out of 10 websites analyzed. Continue with abc and xyz\",\n\"next_goal\": \"What needs to be done with the next immediate action\"},\n\"action\":[{\"one_action_name\": {// action-specific parameter}}, // ... more actions in sequence]}\n\n2. ACTIONS: You can specify multiple actions in the list to be executed in sequence. But always specify only one action name per item. Use maximum {max_actions} actions per sequence.\nCommon action sequences:\n- Form filling: [{\"input_text\": {\"index\": 1, \"text\": \"username\"}}, {\"input_text\": {\"index\": 2, \"text\": \"password\"}}, {\"click_element\": {\"index\": 3}}]\n- Navigation and extraction: [{\"go_to_url\": {\"url\": \"https://example.com\"}}, {\"extract_content\": {\"goal\": \"extract the names\"}}]\n- Actions are executed in the given order\n- If the page changes after an action, the sequence is interrupted and you get the new state.\n- Only provide the action sequence until an action which changes the page state significantly.\n- Try to be efficient, e.g. fill forms at once, or chain actions where nothing changes on the page\n- only use multiple actions if it makes sense.\n\n3. ELEMENT INTERACTION:\n- Only use indexes of the interactive elements\n- Elements marked with \"[]Non-interactive text\" are non-interactive\n\n4. NAVIGATION & ERROR HANDLING:\n- If no suitable elements exist, use other functions to complete the task\n- If stuck, try alternative approaches - like going back to a previous page, new search, new tab etc.\n- Handle popups/cookies by accepting or closing them\n- Use scroll to find elements you are looking for\n- If you want to research something, open a new tab instead of using the current tab\n- If captcha pops up, try to solve it - else try a different approach\n- If the page is not fully loaded, use wait action\n\n5. TASK COMPLETION:\n- Use the done action as the last action as soon as the ultimate task is complete\n- Dont use \"done\" before you are done with everything the user asked you, except you reach the last step of max_steps. \n- If you reach your last step, use the done action even if the task is not fully finished. Provide all the information you have gathered so far. If the ultimate task is completly finished set success to true. If not everything the user asked for is completed set success in done to false!\n- If you have to do something repeatedly for example the task says for \"each\", or \"for all\", or \"x times\", count always inside \"memory\" how many times you have done it and how many remain. Don't stop until you have completed like the task asked you. Only call done after the last step.\n- Don't hallucinate actions\n- Make sure you include everything you found out for the ultimate task in the done text parameter. Do not just say you are done, but include the requested information of the task. \n\n6. VISUAL CONTEXT:\n- When an image is provided, use it to understand the page layout\n- Bounding boxes with labels on their top right corner correspond to element indexes\n\n7. Form filling:\n- If you fill an input field and your action sequence is interrupted, most often something changed e.g. suggestions popped up under the field.\n\n8. Long tasks:\n- Keep track of the status and subresults in the memory. \n\n9. Extraction:\n- If your task is to find information - call extract_content on the specific pages to get and store the information.\nYour responses must be always JSON with the specified format. ",
      "role": "system"
    },
    {
      "content": "Context for the taskAvailable actions: Complete task - with return text and if the task is finished (success=True) or not yet  completly finished (success=False), because last step is reached: \n{done: {'text': {'type': 'string'}, 'success': {'type': 'boolean'}}}\nSearch the query in Google in the current tab, the query should be a search query like humans search in Google, concrete and not vague or super long. More the single most important items. input example: {\"query\": \"your words\"}: \n{search_google: {'query': {'type': 'string'}}}\nNavigate to URL in the current tab: \n{go_to_url: {'url': {'type': 'string'}}}\nGo back: \n{go_back: {}}\nWait for x seconds default 3: \n{wait: {'seconds': {'default': 3, 'type': 'integer'}}}\nClick element: \n{click_element: {'index': {'type': 'integer'}, 'xpath': {'anyOf': [{'type': 'string'}, {'type': 'null'}], 'default': None}}}\nInput text into a input interactive element: \n{input_text: {'index': {'type': 'integer'}, 'text': {'type': 'string'}, 'xpath': {'anyOf': [{'type': 'string'}, {'type': 'null'}], 'default': None}}}\nSwitch tab: \n{switch_tab: {'page_id': {'type': 'integer'}}}\nOpen url in new tab: \n{open_tab: {'url': {'type': 'string'}}}\nExtract page content to retrieve specific information from the page, e.g. all company names, a specifc description, all information about, links with companies in structured format or simply links: \n{extract_content: {'goal': {'type': 'string'}}}\nScroll down the page by pixel amount - if no amount is specified, scroll down one page: \n{scroll_down: {'amount': {'anyOf': [{'type': 'integer'}, {'type': 'null'}], 'default': None}}}\nScroll up the page by pixel amount - if no amount is specified, scroll up one page: \n{scroll_up: {'amount': {'anyOf': [{'type': 'integer'}, {'type': 'null'}], 'default': None}}}\nSend strings of special keys like Escape,Backspace, Insert, PageDown, Delete, Enter, Shortcuts such as `Control+o`, `Control+Shift+T` are supported as well. This gets used in keyboard.press. : \n{send_keys: {'keys': {'type': 'string'}}}\nIf you dont find something which you want to interact with, scroll to it: \n{scroll_to_text: {'text': {'type': 'string'}}}\nGet all options from a native dropdown: \n{get_dropdown_options: {'index': {'type': 'integer'}}}\nSelect dropdown option for interactive element index by the text of the option you want to select: \n{select_dropdown_option: {'index': {'type': 'integer'}, 'text': {'type': 'string'}}}Your ultimate task is: \"\"\"打开Bing搜索 搜索一下一招大模型的信息，多看几篇网页，总结一下重要信息\"\"\". If you achieved your ultimate task, stop everything and use the done action in the next step to complete the task. If not, continue as usual.Example output:Your ultimate task is: \"\"\"打开Bing搜索 搜索一下一招大模型的信息，多看几篇网页，总结一下重要信息\"\"\". If you achieved your ultimate task, stop everything and use the done action in the next step to complete the task. If not, continue as usual.Example output:Your ultimate task is: \"\"\"打开Bing搜索 搜索一下一招大模型的信息，多看几篇网页，总结一下重要信息\"\"\". If you achieved your ultimate task, stop everything and use the done action in the next step to complete the task. If not, continue as usual.Example output:Your ultimate task is: \"\"\"打开Bing搜索 搜索一下一招大模型的信息，多看几篇网页，总结一下重要信息\"\"\". If you achieved your ultimate task, stop everything and use the done action in the next step to complete the task. If not, continue as usual.Example output:",
      "role": "user"
    },
    {
      "content": "[{\"name\": \"AgentOutput\", \"args\": {\"current_state\": {\"evaluation_previous_goal\": \"Success - I opend the first page\", \"memory\": \"Starting with the new task. I have completed 1/10 steps\", \"next_goal\": \"Click on company a\"}, \"action\": [{\"click_element\": {\"index\": 0}}]}, \"id\": \"1\", \"type\": \"tool_call\"}]",
      "role": "assistant"
    },
    {
      "content": "Browser started[Your task history memory starts here]",
      "role": "user"
    },
    {
      "content": "[{\"name\": \"AgentOutput\", \"args\": {\"current_state\": {\"evaluation_previous_goal\": \"Unknown - Analyze the current elements and the image to understand the starting point\", \"memory\": \"Starting with the new task. I have completed 0/10 steps. Need to open Bing search and perform the search query.\", \"next_goal\": \"Navigate to Bing search\"}, \"action\": [{\"go_to_url\": {\"url\": \"https://www.bing.com\"}}]}, \"id\": \"2\", \"type\": \"tool_call\"}]",
      "role": "assistant"
    },
    {
      "content": "Action result: 🔗  Navigated to https://www.bing.com",
      "role": "user"
    },
    {
      "content": "[{\"name\": \"AgentOutput\", \"args\": {\"current_state\": {\"evaluation_previous_goal\": \"Success - Successfully navigated to Bing search page\", \"memory\": \"Starting with the new task. I have completed 1/10 steps. I have navigated to Bing search and need to perform the search query.\", \"next_goal\": \"Perform the search query for '\\u4e00\\u62db\\u5927\\u6a21\\u578b'\"}, \"action\": [{\"input_text\": {\"index\": 11, \"text\": \"\\u4e00\\u62db\\u5927\\u6a21\\u578b\"}}, {\"click_element\": {\"index\": 12}}, {\"wait\": {\"seconds\": 3}}]}, \"id\": \"3\", \"type\": \"tool_call\"}]",
      "role": "assistant"
    },
    {
      "content": "Action result: ⌨️  Input 一招大模型 into index 11Action result: Something new appeared after action 1 / 3",
      "role": "user"
    },
    {
      "content": "[{\"name\": \"AgentOutput\", \"args\": {\"current_state\": {\"evaluation_previous_goal\": \"Success - Successfully navigated to Bing search page and performed the search query.\", \"memory\": \"Starting with the new task. I have completed 1/10 steps. I have navigated to Bing search and performed the search query. Need to click on the search results to view more information.\", \"next_goal\": \"Click on the first search result to view the webpage\"}, \"action\": [{\"click_element\": {\"index\": 16}}, {\"wait\": {\"seconds\": 3}}, {\"extract_content\": {\"goal\": \"extract information about \\u4e00\\u62db\\u5927\\u6a21\\u578b\"}}]}, \"id\": \"4\", \"type\": \"tool_call\"}]",
      "role": "assistant"
    },
    {
      "content": "Action result: 🖱️  Clicked button with index 16: Action result: 🕒  Waiting for 3 secondsAction result: 📄  Extracted from page\n: <think>\n好，我现在需要处理用户的查询，目标是从给定的页面内容中提取关于“一招大模型”的信息，并以JSON格式返回。首先，我得仔细阅读用户提供的页面内容，理解其中的结构和信息分布。\n\n页面看起来像是一个搜索结果页面，包含多个搜索结果条目，每个条目都有标题、来源和简要描述。此外，还有图片搜索结果和其他相关搜索链接。我的任务是从这些内容中提取所有与“一招大模型”相关的信息。\n\n首先，我会浏览每个搜索结果条目，看看它们是否提到了“一招大模型”。例如，第一个结果来自新浪财经，提到招商银行提出了“一招”大模型，这是一次技术创新，也是服务模式的创新。这表明“一招大模型”是招商银行开发的，用于金融领域的AI模型。\n\n接下来，第二个结果是图片搜索，显示了多个相关图片，但可能对提取文本信息帮助不大，所以可以暂时忽略。第三个结果来自ModelScope社区，详细说明了“一招”大模型是由招商银行与华为、智谱AI合作开发的，使用全国产设备和框架，专注于金融领域。这部分信息非常重要，因为它提供了开发背景和合作伙伴。\n\n第四个到第十个结果来自不同的新闻网站，如网易、新浪财经、亿欧、搜狐和百家号，都报道了招商银行申请了“一招大模型”的商标，涉及金融创新和AI技术的应用。这些信息补充了“一招大模型”的商标注册情况和其在金融领域的应用前景。\n\n在提取信息时，我需要确保涵盖以下几点：\n1. “一招大模型”的定义和用途。\n2. 开发背景，包括合作方和开发环境。\n3. 商标注册情况及其意义。\n4. 该模型对金融行业的影响。\n\n此外，我还需要注意信息的来源是否可靠，比如是否来自官方渠道或知名新闻网站。例如，新浪财经和搜狐的报道提供了详细的背景信息，而ModelScope社区则提供了技术细节。\n\n在整理信息时，我会将每个关键点分类，如定义、开发背景、商标注册和影响，确保结构清晰。同时，要避免重复信息，比如多个来源都提到商标注册，但具体内容相似，可以合并描述。\n\n最后，我会将提取的信息组织成JSON格式，确保每个字段准确无误，并且信息全面。这样，用户就能清楚地了解“一招大模型”的各个方面，包括其开发、应用和影响。\n</think>\n\n```json\n{\n  \"extracted_information\": [\n    {\n      \"title\": \"一招大模型\",\n      \"description\": \"一招大模型是由招商银行股份有限公司申请注册的商标，主要用于金融领域的人工智能技术应用。该模型旨在推动银行业服务模式的创新，标志着银行行业将与人工智能深度融合，进入一个全新的智能化服务时代。\",\n      \"source\": \"新浪财经网\",\n      \"date\": \"2024年12月12日\"\n    },\n    {\n      \"title\": \"一招大模型的商标申请\",\n      \"description\": \"招商银行股份有限公司申请注册了“一招（YiZhao）大模型”和“招商银行一招（YiZhao）大模型”商标，国际分类为广告销售、科学仪器等。这一举措表明招商银行在人工智能领域的战略布局。\",\n      \"source\": \"新浪财经\",\n      \"date\": \"2024年12月11日\"\n    },\n    {\n      \"title\": \"一招大模型的技术背景\",\n      \"description\": \"一招大模型是由招商银行联合华为、智谱AI，在全国产的设备与训练框架下训练形成的金融领域大语言模型系列。该模型首先在大规模高质量数据上进行预训练，再通过高质量金融数据进行后期优化。\",\n      \"source\": \"ModelScope 魔搭社区\",\n      \"date\": \"2024年12月\"\n    },\n    {\n      \"title\": \"一招大模型的行业影响\",\n      \"description\": \"一招大模型的提出不仅是技术创新，更是对银行业服务模式的一次大胆创新。它标志着银行行业将与人工智能深度融合，进入一个全新的智能化服务时代。\",\n      \"source\": \"网易\",\n      \"date\": \"2024年12月12日\"\n    },\n    {\n      \"title\": \"一招大模型的商标申请\",\n      \"description\": \"招商银行股份有限公司申请注册了“一招大模型”及相关商标，旨在推动人工智能技术在金融领域的应用与发展。\",\n      \"source\": \"亿欧\",\n      \"date\": \"2024年12月11日\"\n    }\n  ],\n  \"related_searches\": [\n    \"一招大模型商标\",\n    \"招行一招大模型\",\n    \"招商银行一招大模型\",\n    \"招行大模型\",\n    \"招商银行申请大模型\",\n    \"招商银行大模型\"\n  ]\n}\n```\n\n[Task history memory ends]\n[Current state starts here]\nThe following is one-time information - if you need to remember it write it to memory:\nCurrent url: https://www.bing.com/search?q=%e4%b8%80%e6%8b%9b%e5%a4%a7%e6%a8%a1%e5%9e%8b&qs=UT&pq=%e4%b8%80%e6%8b%9b%e5%a4%a7%e6%a8%a1%e5%9e%8b&sc=1-5&cvid=90B5926ACCCE4AF78697D50732768863&FORM=QBLH&sp=1&lq=0\nAvailable tabs:\n[TabInfo(page_id=0, url='https://www.bing.com/search?q=%e4%b8%80%e6%8b%9b%e5%a4%a7%e6%a8%a1%e5%9e%8b&qs=UT&pq=%e4%b8%80%e6%8b%9b%e5%a4%a7%e6%a8%a1%e5%9e%8b&sc=1-5&cvid=90B5926ACCCE4AF78697D50732768863&FORM=QBLH&sp=1&lq=0', title='一招大模型 - 搜索')]\nInteractive elements from top layer of the current page inside the viewport:\n... 26 pixels above - scroll or extract content to see more ...\n跳至内容\n[0]<a />\n[1]<input 在此处输入你的搜索 — 输入时会显示搜索建议;q;一招大模型;search/>\n[2]<div 语音搜索;button/>\n[3]<div false;使用图像搜索;button/>\n[4]<a “搜索”按钮/>\n[5]<input go;搜索;submit/>\n[6]<a />\n[7]<input submit;登录/>\n[8]<a Microsoft Rewards;false;button/>\n[9]<a 设置和快速链接;false;button/>\n[10]<a 全部/>\n[11]<a 图片/>\n[12]<a 视频/>\n[13]<a 地图/>\n[14]<a 资讯/>\n[15]<a Dropdown Menu;button>更多/>\n[16]<a 工具/>\n约 1,070,000 个结果\n[17]<div />\n[18]<a 新浪财经网/>\n[19]<img />\n[20]<div https://cj.sina.com.cn › articles › view/>\n[21]<h2 />\n[22]<a 招行有“一招（YiZhao）” - 新浪财经网/>\n2024年12月12日\n· 如今，招行提出的“一招”大模型，不仅是一次技术创新，更是对银行业服务模式的一次大胆创新，它标志着银行行业将与人工智能深度融合，进入一个全新的智能化服务时代。\n[23]<a 一招大模型的图片>一招大模型的图片\nbing.com › images/>\n[24]<div 单击以向右滚动;button/>\n[25]<div listitem>finance.sina.com.cn/>\n[26]<a 搜索 一招大模型 的图像. finance.sina.com.cn. 550 × 170 · png/>\n[27]<img />\n[28]<a 招商银行申请大模型商标：一招大模型_新浪财经_新浪网/>\n[29]<div listitem>zhuanzhi.ai/>\n[30]<a 搜索 一招大模型 的图像. www.zhuanzhi.ai. 1080 × 608 · jpeg/>\n[31]<img />\n[32]<a 【ChatGPT系列报告】国内大模型概览，42页ppt - 专知VIP/>\n[33]<div listitem>sohu.com/>\n[34]<a 搜索 一招大模型 的图像. www.sohu.com. 1273 × 806 · jpeg/>\n[35]<img />\n[36]<a 招商银行申请 “一招（YiZhao）大模型”商标-北京四海龙知识产权_金融_为客户提供_人工智能/>\n招商银行申请 “一招（YiZhao）大模型”商标-北京四海龙知识产权_金融_为客户提供_人工智能\n什么是大模型？（超详细）大模型从入门到精通，看这一篇就够了-CSDN博客\n查看全部\n[37]<a 查看所有图像/>\n[38]<div />\n[39]<a ModelScope 魔搭社区/>\n[40]<img />\n[41]<div https://www.modelscope.cn › models › CMB_AILab › summary/>\n[42]<h2 />\n[43]<a 魔搭社区/>\n一招（YiZhao)是招商银行联合华为、智谱AI，在全国产的设备与训练框架下训练形成的金融领域大语言模型系列。 我们首先在大规模高质量数据上进行预训练，再通过高质量金融数据进行后期 …\n[44]<div />\n[45]<a 网易/>\n[46]<img />\n[47]<div https://www.163.com › dy › article/>\n[48]<h2 />\n[49]<a 招行有“一招（YiZhao）”|风控|金融|银行业 ... - 网易/>\n2024年12月12日\n· 如今，招行提出的“一招”大模型，不仅是一次技术创新，更是对银行业服务模式的一次大胆创新，它标志着银行行业将与人工智能深度融合，进入一个全新的智能化服务时代。\n[50]<div />\n[51]<a 新浪财经/>\n[52]<img />\n[53]<div https://finance.sina.com.cn › tob/>\n[54]<h2 />\n[55]<a 招商银行申请大模型商标：一招大模型 - 新浪财经/>\n新浪科技讯12月11日上午消息，天眼查知识产权信息显示，近日，招商银行股份有限公司申请注册“一招（YiZhao）大模型”“招商银行一招（YiZhao）大 ...\n[56]<div />\n[57]<a 亿欧/>\n[58]<img />\n[59]<div https://www.iyiou.com › data/>\n[60]<h2 />\n[61]<a 招商银行申请注册“一招大模型”商标，聚焦人工智能领域发展/>\n2024年12月11日\n· 招商银行股份有限公司近日申请注册“一招大模型”及相关商标，旨在推动人工智能技术在金融领域的应用与发展。 近日，根据天眼查知识产权信息显示，招商银行股份有限公 …\n[62]<div />\n[63]<a 搜狐/>\n[64]<img />\n[65]<div https://www.sohu.com/>\n[66]<h2 />\n[67]<a 招商银行申请‘一招大模型’商标：金融创新的未来或将从此开启！/>\n2024年12月11日\n· 近日，招商银行股份有限公司在知识产权领域迈出了新的一步，成功申请了‘一招（YiZhao）大模型’及‘招商银行一招（YiZhao）大模型’的商标。 这一消息于12月11日披露，引 …\n[68]<div />\n[69]<a 百家号/>\n[70]<img />\n[71]<div https://baijiahao.baidu.com/>\n[72]<h2 />\n[73]<a 招商银行布局AI，申请“一招大模型”商标待审 - 百家号/>\n2024年12月11日\n· 近期，招商银行股份有限公司在知识产权保护方面有新动作，据天眼查平台披露的信息，该行已正式提交了两项商标申请，分别为“一招（YiZhao）大模型”与“招商银行一 …\n[74]<div />\n[75]<a 搜狐/>\n[76]<img />\n[77]<div https://www.sohu.com/>\n[78]<h2 />\n[79]<a 招商银行再出大招，申请AI商标意在金融科技新潮流_YiZhao_尖 …/>\n[80]<div />\n[81]<a 百家号/>\n[82]<img />\n[83]<div https://baijiahao.baidu.com/>\n[84]<h2 />\n[85]<a 招商银行申请大模型商标：一招大模型 - 百家号/>\n[86]<div />\n[87]<a 腾讯网/>\n[88]<img />\n[89]<div https://news.qq.com › rain/>\n[90]<h2 />\n[91]<a 招商银行注册“一招大模型”商标 - 腾讯网/>\n[92]<a 一招大模型商标/>\n[93]<a 招行一招大模型/>\n[94]<a 招商银行一招大模型/>\n[95]<a 招行大模型/>\n[96]<a 招商银行申请大模型/>\n[97]<a 招商银行大模型/>\n[98]<a 招商银行大模型商标/>\n[99]<a 招商银行商标一招/>\n[100]<a 第 1 页>1/>\n[101]<a 第 2 页>2/>\n[102]<a 第 3 页>3/>\n[103]<a 第 4 页>4/>\n[104]<a 下一页/>\n相关搜索\n[105]<a 一招大模型商标/>\n[106]<a 招行一招大模型/>\n[107]<a 招商银行一招大模型/>\n[108]<a 招行大模型/>\n[109]<a 招商银行申请大模型/>\n[110]<a 招商银行大模型/>\n[111]<a 招商银行大模型商标/>\n[112]<a 招商银行商标一招/>\n[113]<a 隐私声明和 Cookie/>\n[114]<a 法律声明/>\n[115]<a 广告/>\n[116]<a 关于我们的广告/>\n[117]<a 帮助/>\n[118]<a button>反馈/>\n[119]<a 消费者健康隐私/>\n... 1294 pixels below - scroll or extract content to see more ...\nCurrent step: 4/100Current date and time: 2025-03-27 17:53\n",
      "role": "user"
    }
  ],
  "model": "deepseek-r1-distill-qwen-32b",
  "stream": false,
  "temperature": 0
}
```

##### [microsoft/*playwright* 71k star](https://github.com/microsoft/playwright)

***Playwright*** is a framework for Web Testing and Automation. It allows testing Chromium, Firefox and WebKit with a single API.

提供js、python、java等sdk允许不用语言通过playwright api操作浏览器

#### Computer-use

##### [claude-3-5-sonnet-20241022 computer-use-demo](https://github.com/anthropics/anthropic-quickstarts/tree/main/computer-use-demo)

1022发布的开源demo，已经可以完成浏览网页，亚马逊购物，画图等功能。llm支持的actions如下：

<img src="./2025-03-27-16-12-45-image.png" title="" alt="" width="412">

computer-use-demo代码中会loop调用anthropic api并执行tool action直到llm返回中不包含tool action

**"用画图画一只猫"**

![](./2025-03-27-15-45-22-image.png)

##### [OpenAI Computer-use-example](https://platform.openai.com/docs/guides/tools-computer-use)

支持的actions：

```markdown
screenshot
click
double_click
type
keypress
move
scroll
wait
drag
```

![Computer use diagram](https://cdn.openai.com/API/docs/images/cua_diagram.png)

##### [E2B computer-use agent](https://surf.e2b.dev/)

一家为AI提供云虚拟化环境的公司，用开源大模型实现了较好的效果。

<img title="" src="./2025-03-27-11-23-48-image.png" alt="" width="539">

##### [微软 omniParser omniTool](https://github.com/microsoft/OmniParser/)

![](./2025-03-27-15-37-02-image.png)

##### [OS-ATLAS: Foundation Action Model for Generalist GUI Agents](https://arxiv.org/abs/2410.23218)

## MCP及Agent接入

[Introduction - Model Context Protocol](https://modelcontextprotocol.io/introduction)

提供一套标准协议，方便第三方能力(tools, prompts, resources等)快速接入LLM Agents

![](./2025-03-28-09-16-47-image.png)

#### MCP core concepts

![](./2025-03-28-09-22-54-image.png)

official java sdk 

![](./2025-03-28-09-01-24-image.png)

#### MCP的支持及有趣案例

**LLM App**：Claude destop, OpenAI desktop，Cursor, VS code Cline

**开源社区**：langchain，dify

MCP server案例：

建模引擎：blender, unity

![](./2025-03-28-13-40-50-image.png)

办公类：邮件、会议等

游戏类：steam, minecraft

地图类：谷歌、百度、高德地图

Stock类：实时股价、选股

IOT: HomeAssistant，智能家居

#### Open-Source MCP Servers

[GitHub - punkpeye/awesome-mcp-servers: A collection of MCP servers.](https://github.com/punkpeye/awesome-mcp-servers)

[Open-Source MCP servers | Glama](https://glama.ai/mcp/servers)

[MCP.SO](https://mcp.so/)

#### Cline测试

Cline安装配置过程略，mcp server仅安装playwright

###### 测试结果

task prompt：上网搜索一下有关一招大模型的信息，挑几篇网页详细阅读一下，整理出一篇介绍并保存到本地yizhao.md文件，注意标好引用链接地址。

![](./2025-03-28-09-46-40-image.png)

![](./2025-03-28-09-47-23-image.png)

![](./2025-03-28-09-49-17-image.png)

![](./2025-03-28-09-51-33-image.png)

![](./2025-03-28-09-52-06-image.png)

![](./2025-03-28-09-53-16-image.png)

![](./2025-03-28-09-53-54-image.png)

![](./2025-03-28-09-54-36-image.png)

###### cline prompt log

cline systemPrompt 12k tokens，然后loop执行action
[prompt](./cline-exec-req.txt)

##### 行内cline、mcp、大模型测试：

见内网

总行团队一些案例

#### MCP发展

langchain社区：[MCP: Flash in the Pan or Future Standard?](https://blog.langchain.dev/mcp-fad-or-fixture/)
```
