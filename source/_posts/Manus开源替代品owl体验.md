---
title: Manus开源替代品owl体验
date: 2025-03-09 11:44:57
tags: manus, agent, owl
---

# Overview
这篇文章主要用于记录体验owl的情况和遇到的问题。

[OWL](https://github.com/camel-ai/owl)是一个多智能体协作的框架，提供Demo供测试。
其比较突出的成就是**在 GAIA 基准测试中取得 58.18 平均分，在开源框架中排名 🏅️ #1**，请注意，这里限定的是开源框架，而实际上在[GAIA LeaderBorad](https://huggingface.co/spaces/gaia-benchmark/leaderboard)上其排名第三，第一名是Trase Agent v0.3获得70.3的高分。
值得注意的是，博主并没有发现Manus和OpenManus在榜。
![](Gaia.png)

# TL;DR
博主从2025年3月7日晚测试到2025年3月9日中午为止，仍未能跑通一次流程。最长的流程跑了1个半小时，而最终也无法得到结果。

OWL**也许**很强，但是作为一名普通用户，在使用体验上是比较糟糕的，报错了难以快速解决，一个问题卡住几十分钟甚至一个小时。这可能导致用户转向Manus/OpenManus或者其他后起之秀。

在文档指引上，OWL目前也是很欠缺，目前只给了一个`.env_template`和几个`demo.py`。Readme中有一点点指引，但也是杯水车薪。博主参加了2025年3月8日Camel.ai的OWL开发者大会直播，根据会上提到的，这可能是由于其人手不足的缘故，毕竟目前他们只有一个全职和几个兼职参与。

# 安装准备
请参考官方文档进行环境准备。
## 建议
- 如果使用OpenAI，可以使用OpenRouter，支持支付宝充值，然后需要稍微注意一下额度，因为4o的收费还是不便宜的。
- 建议把所有API_key都配置上，因为按照目前owl的文档水平，你不清楚哪些是必须哪些不是。
- 使用哪个平台时，记得充值，否则没有额度的情况下无法调通。而报错信息也不会友好地提示你缺少额度。

# 体验: 查询一年来Rust在大模型上的生态和发展
我们将对不同的demo给出同样的提问：**一年来Rust在大模型上的生态和发展如何，给我一个.**

## 失败案例1：DeepSeek 
[运行过程](deepseek执行过程.txt)

2025年3月9日的早上，owl发布了deepseek的demo，而博主接入后，从10:35开始运行，直到12:14因为异常结束流程。最终也未能得到结果。最后消费金额¥1.85
```log
(owl) D:\proj\owl>python owl/run_deepseek_example.py
2025-03-09 10:35:52,806 - camel - INFO - Camel library logging has been configured.
2025-03-09 10:36:00,783 - httpx - INFO - HTTP Request: POST https://api.deepseek.com/chat/completions "HTTP/1.1 200 OK"
2025-03-09 10:36:12,993 - httpx - INFO - HTTP Request: POST https://api.deepseek.com/chat/completions "HTTP/1.1 200 OK"
2025-03-09 10:36:20,929 - httpx - INFO - HTTP Request: POST https://api.deepseek.com/chat/completions "HTTP/1.1 200 OK"
2025-03-09 10:36:29.866 | DEBUG    | camel.toolkits.search_toolkit:search_google:370 - Calling search_google function with query: University of Leicester paper Can Hiccup Supply Enough Fish to Maintain a Dragon’s Diet

....

2025-03-09 12:14:48,540 - httpx - INFO - HTTP Request: POST https://api.deepseek.com/chat/completions "HTTP/1.1 400 Bad Request"
2025-03-09 12:14:48,541 - camel.models.model_manager - ERROR - Error processing with model: <camel.models.deepseek_model.DeepSeekModel object at 0x0000023A4E321ED0>
2025-03-09 12:14:48.542 | ERROR    | camel.agents.chat_agent:_step_model_response:1000 - An error occurred while running model deepseek-chat, index: 0
Traceback (most recent call last):
  File "D:\proj\owl\owl\camel\toolkits\function_tool.py", line 349, in __call__
    result = self.func(*args, **kwargs)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\proj\owl\owl\camel\toolkits\search_toolkit.py", line 732, in web_search
    resp = search_agent.step(prompt)
           ^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\proj\owl\owl\camel\agents\chat_agent.py", line 672, in step
    ) = self._step_model_response(openai_messages, num_tokens)
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\proj\owl\owl\camel\agents\chat_agent.py", line 1008, in _step_model_response
    raise ModelProcessingError(
camel.models.model_manager.ModelProcessingError: Unable to process messages: none of the provided models run succesfully.

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\proj\owl\owl\run_deepseek_example.py", line 77, in <module>
    answer, chat_history, token_count = run_society(society)
                                        ^^^^^^^^^^^^^^^^^^^^
  File "D:\proj\owl\owl\utils\enhanced_role_playing.py", line 412, in run_society
    assistant_response, user_response = society.step(input_msg)
                                        ^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\proj\owl\owl\utils\enhanced_role_playing.py", line 253, in step
    assistant_response = self.assistant_agent.step(modified_user_msg)
                         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\proj\owl\owl\camel\agents\chat_agent.py", line 705, in step
    self._step_tool_call_and_update(response)
  File "D:\proj\owl\owl\camel\agents\chat_agent.py", line 882, in _step_tool_call_and_update
    self.step_tool_call(response)
  File "D:\proj\owl\owl\camel\agents\chat_agent.py", line 1294, in step_tool_call
    result = tool(**args)
             ^^^^^^^^^^^^
  File "D:\proj\owl\owl\camel\toolkits\function_tool.py", line 352, in __call__
    raise ValueError(
ValueError: Execution of function web_search failed with arguments () and {'question': 'Rust ecosystem and development in large models 2023'}. Error: Unable to process messages: none of the provided models run succesfully.
```

由于时间有限，博主看了一小段执行过程，发现其中出现了大量重复的内容，有可能是存在Bug导致循环调用该工具，这才导致长时间运行而没有出结果。
```log
2025-03-09 10:43:24.581 | DEBUG    | camel.toolkits.search_toolkit:search_google:370 - Calling search_google function with query: Rust ecosystem large models 2023
2025-03-09 10:43:25.131 | DEBUG    | camel.toolkits.search_toolkit:search_google:454 - search result: [{'result_id': 1, 'title': 'Rust + WebAssembly: Building Infrastructure for Large Language ...', 'description': 'Rust + WebAssembly: Building Infrastructure for Large Language Model Ecosystems. Oct 30, 2023 • 12 minutes to read. Follow · Share to Twitter.', 'long_description': 'N/A', 'url': 'https://www.secondstate.io/articles/infra-for-llms/'}, {'result_id': 2, 'title': "I can't decide: Rust or C++ : r/rust", 'description': "Apr 24, 2023 ... ... models on microprocessors and ... rust ecosystem. Upvote 39. Downvote Reply reply. Award Share. gopher_protocol. • 2y ago. Though I'm a big Rust\xa0...", 'long_description': 'Posted by u/cconnection - 281 votes and 252 comments', 'url': 'https://www.reddit.com/r/rust/comments/12xgxfa/i_cant_decide_rust_or_c/'}, {'result_id': 3, 'title': "Rust beyond CPU's (for HW such as TPUs, GPUs, accelerators etc ...", 'description': 'Nov 29, 2023 ... November 29, 2023, 10:25am 1. So, I was wondering if there are projects (or communities) within the larger Rust ecosystem working on bringing\xa0...', 'long_description': "So, I was wondering if there are projects (or communities) within the larger Rust ecosystem working on bringing Rust to a more diverse range of hardware, such as TPUs, GPUs, or custom accelerators. I primarily work in the systems domain (embedded + high performance), but I've noticed an increasing shift of compute towards specialized hardware—things that are not CPUs—for various reasons like parallel processing, efficiency, and dedicated memory architectures.  I believe this trend is partly due ...", 'url': 'https://users.rust-lang.org/t/rust-beyond-cpus-for-hw-such-as-tpus-gpus-accelerators-etc/103224'}, {'result_id': 4, 'title': 'Considering C++ over Rust. : r/cpp', 'description': "Sep 4, 2023 ... Low entry in rust ecosystem is big plus. In my opinion it's easier ... model? Nope, all basic integer types are defined by the compiler\xa0...", 'long_description': 'Posted by u/isht_0x37 - 349 votes and 435 comments', 'url': 'https://www.reddit.com/r/cpp/comments/16a0c9x/considering_c_over_rust/'}, {'result_id': 5, 'title': "rtyler's blog", 'description': 'Jul 9, 2024 ... Large language models (LLMs) seem to only be good at two things ... Dependency management in the Rust ecosystem is fairly mature from\xa0...', 'long_description': 'a moderately technical blog', 'url': 'https://brokenco.de/page2/'}, {'result_id': 6, 'title': 'rustformers/llm: [Unmaintained, see README] An ... - GitHub', 'description': "Jun 24, 2024 ... llm is an ecosystem of Rust libraries for working with large language models - it's built on top of the fast, efficient GGML library for machine learning.", 'long_description': '[Unmaintained, see README] An ecosystem of Rust libraries for working with large language models - rustformers/llm', 'url': 'https://github.com/rustformers/llm'}]
```

## 失败案例2：使用OpenAI体验失败
OpenAI对应的demo是`owl/run.py`，这个用例很好，好就好在你OpenRouter的base_url和key都配置好了，一运行直接报错，直接省去等待时间。
```log
(owl) D:\proj\owl>python owl/run.py
2025-03-08 22:50:15,694 - camel - INFO - Camel library logging has been configured.
2025-03-08 22:50:24,320 - camel.toolkits.video_downloader_toolkit - INFO - Video will be downloaded to C:\Users\qbug\AppData\Local\Temp\tmpjidewpup
2025-03-08 22:50:24,323 - camel.toolkits.video_analysis_toolkit - INFO - Video will be downloaded to C:\Users\qbug\AppData\Local\Temp\tmpjidewpup
2025-03-08 22:50:28,522 - httpx - INFO - HTTP Request: POST https://openrouter.ai/api/v1/chat/completions "HTTP/1.1 200 OK"
2025-03-08 22:50:31,251 - httpx - INFO - HTTP Request: POST https://openrouter.ai/api/v1/chat/completions "HTTP/1.1 200 OK"
Traceback (most recent call last):
  File "D:\proj\owl\owl\run.py", line 78, in <module>
    answer, chat_history, token_count = run_society(society)
                                        ^^^^^^^^^^^^^^^^^^^^
  File "D:\proj\owl\owl\utils\enhanced_role_playing.py", line 412, in run_society
    assistant_response, user_response = society.step(input_msg)
                                        ^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\proj\owl\owl\utils\enhanced_role_playing.py", line 253, in step
    assistant_response = self.assistant_agent.step(modified_user_msg)
                         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\proj\owl\owl\camel\agents\chat_agent.py", line 672, in step
    ) = self._step_model_response(openai_messages, num_tokens)
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\proj\owl\owl\camel\agents\chat_agent.py", line 1021, in _step_model_response
    self.handle_batch_response(response)
  File "D:\proj\owl\owl\camel\agents\chat_agent.py", line 1119, in handle_batch_response
    for choice in response.choices:
TypeError: 'NoneType' object is not iterable
Exception ignored in: <function VideoDownloaderToolkit.__del__ at 0x00000274FF898360>
Traceback (most recent call last):
  File "D:\proj\owl\owl\camel\toolkits\video_downloader_toolkit.py", line 116, in __del__
ImportError: sys.meta_path is None, Python is likely shutting down
```

## 失败案例3：使用qwen体验失败
owl提供了一个使用qwen模型的`owl/run_qwen_demo.py`可以直接使用，但是在实际使用过程中，会遇到非常多问题。具体可以查看[执行过程](qwen执行过程.txt)

### 问题1：默认使用的qwq-32b无法调通
如果你直接使用提供的demo，那大概率会遇到这个问题:
```log
2025-03-08 21:12:02.869 | ERROR    | camel.agents.chat_agent:_step_model_response:1000 - An error occurred while running model qwq-32b, index: 0
```

如果你往下debug到http request的结果，可以发现响应报错大概意思是：**qwq-32b只支持流式传输**

这个时候我们需要把模型改为`qwen-plus-latest`才可以正常使用，由此已经可以看出owl在用户体验这块还是有待提升的。

而改完模型后，紧接着又出现了问题2

### 问题2: 浏览器工具出错
在使用qwen，并且改为支持vl的plus模型后，可以看到Terminal里可以正常执行了，可以看到下面的日志里提到已经在调用`search toolbit`来做一些查询动作了
```log
2025-03-08 21:14:32.756 | INFO     | utils.enhanced_role_playing:run_society:428 - Round #0 user_response:
 Instruction: Use the search toolkit to find recent articles and reports about Rust's ecosystem and development in large models over the past year.

            Here are auxiliary information about the overall task, which may help you understand the intent of the current task:
            <auxiliary_information>
            一年来Rust在大模型上的生态和发展如何，给我一个.
            </auxiliary_information>
            If there are available tools and you want to call them, never say 'I will ...', but first call the tool and reply based on tool call's result, and tell me which tool you have called.
```

然后经过接近40分钟的等待，诶，失败了。但是过程中有一些Planing, Tool Call还是正常的，只是似乎在最后调用浏览器的时候失败了，最终导致整个过程失败，所以我们苦等40分钟，最终结果只得到了中间过程。

```log
2025-03-08 21:53:38.593 | DEBUG    | camel.toolkits.web_toolkit:browser_simulation:1047 - Detailed plan: ### Restated Task:
The task is to visit the webpage `https://www.secondstate.io/articles/infra-for-llms/` and extract relevant information about Rust's ecosystem and its development in large language models (LLMs) over the past year. The process should be considered as a Partially Observable Markov Decision Process (POMDP), meaning that not all information may be directly observable, and decisions must be made based on partial information.

### Detailed Plan:

#### Step 1: Access the Webpage
- **Action:** Navigate to the URL `https://www.secondstate.io/articles/infra-for-llms/`.
- **Observation:** Confirm that the page has loaded successfully and identify the main sections of the article.
- **Decision Point:** If the page does not load or if there are issues accessing the content, consider alternative actions such as refreshing the page or checking for network connectivity.

#### Step 2: Scan the Article for Relevant Sections
- **Action:** Perform a quick scan of the article by looking at headings, subheadings, and any highlighted text that might relate to Rust's ecosystem and LLMs.
- **Observation:** Identify sections that mention Rust, its ecosystem, and developments related to LLMs.
- **Decision Point:** If no relevant sections are immediately apparent, consider using the browser's search function (Ctrl+F) to look for keywords like "Rust," "ecosystem," "development," and "large models."

#### Step 3: Extract Information
- **Action:** Carefully read through the identified sections and extract pertinent information. Look for details about how Rust has been used in developing LLMs, any new tools or libraries introduced, performance improvements, community contributions, and other relevant advancements.
- **Observation:** Take notes of specific points, dates, statistics, and quotes that highlight Rust's role and progress in the field of LLMs over the past year.
- **Decision Point:** If the information is scattered or not clearly presented, decide whether to continue searching within the article or to seek additional sources for more comprehensive data.

#### Step 4: Summarize Findings
- **Action:** Compile the extracted information into a coherent summary. Ensure that the summary covers the key aspects of Rust's ecosystem and its development in LLMs over the past year.
- **Observation:** Review the summary to ensure it accurately reflects the findings from the article and provides a clear understanding of Rust's contributions to LLMs.
- **Decision Point:** If the summary seems incomplete or lacks depth, revisit the article or consider consulting other resources to fill in any gaps.

#### Step 5: Validate and Report
- **Action:** Double-check the accuracy of the summarized information against the original article. Prepare the final report or presentation of the findings.
- **Observation:** Ensure that all claims are supported by evidence from the article and that the report is well-organized and easy to understand.
- **Decision Point:** Before finalizing the report, consider having it reviewed by a peer or expert to catch any potential errors or omissions.

By following this plan, we can systematically approach the task while accounting for the uncertainties inherent in a POMDP framework. Each step involves making decisions based on the available observations, which allows for flexibility and adaptability throughout the process.
Traceback (most recent call last):
  File "D:\proj\owl\owl\camel\toolkits\function_tool.py", line 349, in __call__
    result = self.func(*args, **kwargs)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\proj\owl\owl\camel\toolkits\web_toolkit.py", line 1049, in browser_simulation
    self.browser.init()
  File "D:\proj\owl\owl\camel\toolkits\web_toolkit.py", line 305, in init
    self.browser = self.playwright.chromium.launch(headless=self.headless)      # launch the browser, if the headless is False, the browser will be displayed
                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\qbug\miniconda3\envs\owl\Lib\site-packages\playwright\sync_api\_generated.py", line 14461, in launch
    self._sync(
  File "C:\Users\qbug\miniconda3\envs\owl\Lib\site-packages\playwright\_impl\_sync_base.py", line 104, in _sync
    raise Error("Event loop is closed! Is Playwright already stopped?")
playwright._impl._errors.Error: Event loop is closed! Is Playwright already stopped?

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\proj\owl\owl\run_qwq_demo.py", line 85, in <module>
    answer, chat_history, token_count = run_society(society)
                                        ^^^^^^^^^^^^^^^^^^^^
  File "D:\proj\owl\owl\utils\enhanced_role_playing.py", line 412, in run_society
    assistant_response, user_response = society.step(input_msg)
                                        ^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\proj\owl\owl\utils\enhanced_role_playing.py", line 253, in step
    assistant_response = self.assistant_agent.step(modified_user_msg)
                         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\proj\owl\owl\camel\agents\chat_agent.py", line 705, in step
    self._step_tool_call_and_update(response)
  File "D:\proj\owl\owl\camel\agents\chat_agent.py", line 882, in _step_tool_call_and_update
    self.step_tool_call(response)
  File "D:\proj\owl\owl\camel\agents\chat_agent.py", line 1294, in step_tool_call
    result = tool(**args)
             ^^^^^^^^^^^^
  File "D:\proj\owl\owl\camel\toolkits\function_tool.py", line 352, in __call__
    raise ValueError(
ValueError: Execution of function browser_simulation failed with arguments () and {'start_url': 'https://www.secondstate.io/articles/infra-for-llms/', 'task_prompt': "Visit the link and extract relevant information about Rust's ecosystem and development in large models over the past year."}. Error: Event loop is closed! Is Playwright already stopped?
```