# Agent 系统实战案例详解

> 真实场景的 Agent 系统开发案例，包含完整实现、难点分析和解决方案
> 
> @author erik.zhou

## 📋 目录

- [案例 1：电商智能客服系统](#案例-1电商智能客服系统)
- [案例 2：企业知识库问答系统](#案例-2企业知识库问答系统)
- [案例 3：自动化运维 Agent](#案例-3自动化运维-agent)
- [案例 4：智能数据分析平台](#案例-4智能数据分析平台)
- [案例 5：代码审查与重构助手](#案例-5代码审查与重构助手)
- [案例 6：多语言翻译与本地化系统](#案例-6多语言翻译与本地化系统)
- [案例 7：智能招聘筛选系统](#案例-7智能招聘筛选系统)

---

## 案例 1：电商智能客服系统

### 1.1 业务场景

某电商平台日均咨询量 10 万+，人工客服成本高、响应慢。需要构建智能客服系统：
- 自动回答常见问题（退换货、物流查询等）
- 处理订单相关操作（查询、取消、修改）
- 智能推荐商品
- 复杂问题转人工

### 1.2 系统架构

```
用户输入
    ↓
意图识别 Agent
    ↓
┌─────────┬─────────┬─────────┬─────────┐
│ FAQ     │ 订单    │ 商品    │ 人工    │
│ Agent   │ Agent   │ Agent   │ 转接    │
└─────────┴─────────┴─────────┴─────────┘
    ↓
响应生成
    ↓
用户反馈
```

### 1.3 核心实现

```python
from typing import Dict, Any, List, Optional
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain.tools import Tool
from langchain.vectorstores import Chroma
from langchain.memory import ConversationBufferWindowMemory
from pydantic import BaseModel, Field
import logging
from datetime import datetime
import json

logger = logging.getLogger(__name__)


class IntentClassifier:
    """意图识别器"""
    
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
        
        # 定义意图类型
        self.intents = {
            "faq": ["退货", "换货", "发票", "配送", "支付", "优惠券"],
            "order": ["订单查询", "取消订单", "修改地址", "物流追踪"],
            "product": ["商品推荐", "商品对比", "库存查询", "价格咨询"],
            "complaint": ["投诉", "差评", "质量问题"],
            "human": ["转人工", "人工客服"]
        }
    
    def classify(self, user_input: str) -> Dict[str, Any]:
        """识别用户意图"""
        
        # 使用 LLM 进行意图分类
        prompt = f"""
分析用户输入的意图，从以下类别中选择最匹配的：

意图类别：
- faq: 常见问题咨询（退换货、发票、配送等）
- order: 订单相关操作
- product: 商品相关咨询
- complaint: 投诉建议
- human: 需要人工客服

用户输入：{user_input}

请以 JSON 格式返回：
{{
    "intent": "意图类别",
    "confidence": 0.0-1.0,
    "keywords": ["关键词1", "关键词2"]
}}
"""
        
        try:
            response = self.llm.invoke(prompt)
            result = json.loads(response.content)
            
            logger.info(f"意图识别: {result['intent']}, 置信度: {result['confidence']}")
            return result
            
        except Exception as e:
            logger.error(f"意图识别失败: {str(e)}")
            return {
                "intent": "faq",
                "confidence": 0.5,
                "keywords": []
            }


class OrderService:
    """订单服务（模拟数据库）"""
    
    def __init__(self):
        # 模拟订单数据
        self.orders = {
            "ORD20240301001": {
                "order_id": "ORD20240301001",
                "user_id": "USER001",
                "status": "已发货",
                "items": [
                    {"name": "iPhone 15 Pro", "quantity": 1, "price": 7999}
                ],
                "total": 7999,
                "shipping": {
                    "address": "北京市朝阳区xxx",
                    "tracking_no": "SF1234567890",
                    "carrier": "顺丰速运"
                },
                "created_at": "2024-03-01 10:30:00",
                "estimated_delivery": "2024-03-05"
            },
            "ORD20240302001": {
                "order_id": "ORD20240302001",
                "user_id": "USER001",
                "status": "待支付",
                "items": [
                    {"name": "AirPods Pro", "quantity": 1, "price": 1999}
                ],
                "total": 1999,
                "created_at": "2024-03-02 15:20:00"
            }
        }
    
    def query_order(self, order_id: str) -> Optional[Dict]:
        """查询订单"""
        return self.orders.get(order_id)
    
    def query_user_orders(self, user_id: str, limit: int = 10) -> List[Dict]:
        """查询用户订单列表"""
        user_orders = [
            order for order in self.orders.values()
            if order["user_id"] == user_id
        ]
        return sorted(
            user_orders,
            key=lambda x: x["created_at"],
            reverse=True
        )[:limit]
    
    def cancel_order(self, order_id: str) -> Dict[str, Any]:
        """取消订单"""
        order = self.orders.get(order_id)
        
        if not order:
            return {"success": False, "message": "订单不存在"}
        
        if order["status"] == "已发货":
            return {"success": False, "message": "订单已发货，无法取消"}
        
        if order["status"] == "已完成":
            return {"success": False, "message": "订单已完成，无法取消"}
        
        # 模拟取消操作
        order["status"] = "已取消"
        
        return {
            "success": True,
            "message": f"订单 {order_id} 已成功取消",
            "refund_amount": order["total"]
        }
    
    def track_shipping(self, tracking_no: str) -> Dict[str, Any]:
        """物流追踪"""
        # 模拟物流信息
        tracking_info = {
            "SF1234567890": {
                "carrier": "顺丰速运",
                "status": "运输中",
                "current_location": "北京分拨中心",
                "estimated_delivery": "2024-03-05",
                "history": [
                    {"time": "2024-03-01 14:00", "status": "已揽收", "location": "北京朝阳区"},
                    {"time": "2024-03-01 18:00", "status": "运输中", "location": "北京分拨中心"},
                    {"time": "2024-03-02 08:00", "status": "运输中", "location": "天津分拨中心"}
                ]
            }
        }
        
        return tracking_info.get(tracking_no, {"error": "物流信息不存在"})


class FAQService:
    """常见问题服务"""
    
    def __init__(self):
        self.embeddings = OpenAIEmbeddings()
        
        # 初始化向量数据库
        self.vectorstore = self._init_vectorstore()
    
    def _init_vectorstore(self):
        """初始化 FAQ 向量库"""
        
        # FAQ 数据
        faqs = [
            {
                "question": "如何申请退货？",
                "answer": """退货流程：
1. 登录账户，进入"我的订单"
2. 选择需要退货的订单，点击"申请退货"
3. 选择退货原因并上传凭证（如有质量问题）
4. 提交申请，等待审核
5. 审核通过后，按照提示寄回商品
6. 商品验收后，3-5个工作日退款到账

注意事项：
- 商品需保持原包装完好
- 签收后7天内可申请退货
- 特殊商品（如生鲜、定制品）不支持退货"""
            },
            {
                "question": "退款多久到账？",
                "answer": """退款到账时间：
- 原路退回：3-5个工作日
- 退回银行卡：5-7个工作日
- 退回支付宝/微信：1-3个工作日

如超过预计时间未到账，请联系客服查询。"""
            },
            {
                "question": "如何开具发票？",
                "answer": """发票开具流程：
1. 订单完成后，进入"我的订单"
2. 点击"申请发票"
3. 填写发票信息（个人/企业）
4. 提交申请
5. 电子发票将在1-3个工作日发送到您的邮箱

支持类型：
- 个人发票：普通发票
- 企业发票：增值税普通发票、增值税专用发票"""
            },
            {
                "question": "配送时效是多久？",
                "answer": """配送时效：
- 一线城市：1-2天
- 二三线城市：2-3天
- 偏远地区：3-7天

加急服务：
- 当日达：部分城市支持，需额外支付费用
- 次日达：大部分城市支持

注意：节假日可能延迟1-2天"""
            },
            {
                "question": "支持哪些支付方式？",
                "answer": """支持的支付方式：
1. 在线支付：
   - 支付宝
   - 微信支付
   - 银联在线
   - 信用卡（Visa、MasterCard）

2. 货到付款：
   - 部分地区支持
   - 需额外支付5元手续费

3. 分期付款：
   - 花呗分期
   - 信用卡分期"""
            }
        ]
        
        # 构建文档
        texts = [f"问题：{faq['question']}\n答案：{faq['answer']}" for faq in faqs]
        metadatas = [{"question": faq["question"], "answer": faq["answer"]} for faq in faqs]
        
        # 创建向量库
        vectorstore = Chroma.from_texts(
            texts=texts,
            embedding=self.embeddings,
            metadatas=metadatas,
            collection_name="customer_service_faq"
        )
        
        return vectorstore
    
    def search_faq(self, query: str, top_k: int = 3) -> List[Dict[str, str]]:
        """搜索相关 FAQ"""
        results = self.vectorstore.similarity_search_with_score(query, k=top_k)
        
        faqs = []
        for doc, score in results:
            if score < 0.5:  # 相似度阈值
                faqs.append({
                    "question": doc.metadata["question"],
                    "answer": doc.metadata["answer"],
                    "relevance_score": round(1 - score, 2)  # 转换为相似度分数
                })
        
        return faqs


class ProductService:
    """商品服务"""
    
    def __init__(self):
        # 模拟商品数据
        self.products = {
            "PROD001": {
                "id": "PROD001",
                "name": "iPhone 15 Pro",
                "category": "手机",
                "price": 7999,
                "stock": 100,
                "description": "A17 Pro芯片，钛金属设计",
                "specs": {
                    "屏幕": "6.1英寸",
                    "存储": "256GB",
                    "颜色": "钛金属"
                }
            },
            "PROD002": {
                "id": "PROD002",
                "name": "AirPods Pro",
                "category": "耳机",
                "price": 1999,
                "stock": 50,
                "description": "主动降噪，空间音频",
                "specs": {
                    "降噪": "主动降噪",
                    "续航": "6小时"
                }
            }
        }
    
    def search_products(self, keyword: str, category: Optional[str] = None) -> List[Dict]:
        """搜索商品"""
        results = []
        
        for product in self.products.values():
            # 关键词匹配
            if keyword.lower() in product["name"].lower() or \
               keyword.lower() in product["description"].lower():
                
                # 分类过滤
                if category and product["category"] != category:
                    continue
                
                results.append(product)
        
        return results
    
    def get_product_detail(self, product_id: str) -> Optional[Dict]:
        """获取商品详情"""
        return self.products.get(product_id)
    
    def check_stock(self, product_id: str) -> Dict[str, Any]:
        """检查库存"""
        product = self.products.get(product_id)
        
        if not product:
            return {"available": False, "message": "商品不存在"}
        
        return {
            "available": product["stock"] > 0,
            "stock": product["stock"],
            "message": f"当前库存：{product['stock']} 件"
        }


```



class CustomerServiceAgent:
    """电商智能客服 Agent"""
    
    def __init__(self, user_id: str):
        self.user_id = user_id
        self.llm = ChatOpenAI(model="gpt-4", temperature=0.7)
        
        # 初始化服务
        self.intent_classifier = IntentClassifier()
        self.order_service = OrderService()
        self.faq_service = FAQService()
        self.product_service = ProductService()
        
        # 初始化记忆
        self.memory = ConversationBufferWindowMemory(
            memory_key="chat_history",
            k=5,  # 保留最近5轮对话
            return_messages=True
        )
        
        # 创建工具
        self.tools = self._create_tools()
        
        # 创建 Agent
        self.agent = self._create_agent()
        
        # 会话状态
        self.session_state = {
            "current_order": None,
            "pending_action": None,
            "satisfaction_score": None
        }
    
    def _create_tools(self) -> List[Tool]:
        """创建工具列表"""
        return [
            Tool(
                name="query_order",
                func=self._query_order_tool,
                description="""查询订单信息。
输入格式：订单号（如：ORD20240301001）
返回：订单详情，包括状态、商品、物流等信息"""
            ),
            Tool(
                name="query_my_orders",
                func=self._query_my_orders_tool,
                description="""查询当前用户的订单列表。
输入：无需输入或输入"最近订单"
返回：用户最近的订单列表"""
            ),
            Tool(
                name="cancel_order",
                func=self._cancel_order_tool,
                description="""取消订单。
输入格式：订单号
返回：取消结果和退款信息"""
            ),
            Tool(
                name="track_shipping",
                func=self._track_shipping_tool,
                description="""追踪物流信息。
输入格式：物流单号
返回：物流状态和轨迹"""
            ),
            Tool(
                name="search_faq",
                func=self._search_faq_tool,
                description="""搜索常见问题答案。
输入：问题描述
返回：相关的FAQ答案"""
            ),
            Tool(
                name="search_products",
                func=self._search_products_tool,
                description="""搜索商品。
输入：商品关键词
返回：匹配的商品列表"""
            ),
            Tool(
                name="check_stock",
                func=self._check_stock_tool,
                description="""检查商品库存。
输入：商品ID
返回：库存状态"""
            )
        ]
    
    def _query_order_tool(self, order_id: str) -> str:
        """查询订单工具"""
        order = self.order_service.query_order(order_id.strip())
        
        if not order:
            return f"未找到订单 {order_id}，请检查订单号是否正确"
        
        # 保存当前订单到会话状态
        self.session_state["current_order"] = order_id
        
        # 格式化订单信息
        result = f"""订单信息：
订单号：{order['order_id']}
状态：{order['status']}
下单时间：{order['created_at']}

商品清单：
"""
        for item in order['items']:
            result += f"- {item['name']} x{item['quantity']} ￥{item['price']}\n"
        
        result += f"\n订单总额：￥{order['total']}"
        
        if order['status'] == '已发货' and 'shipping' in order:
            result += f"""

物流信息：
配送地址：{order['shipping']['address']}
物流公司：{order['shipping']['carrier']}
物流单号：{order['shipping']['tracking_no']}
预计送达：{order['estimated_delivery']}"""
        
        return result
    
    def _query_my_orders_tool(self, query: str = "") -> str:
        """查询我的订单工具"""
        orders = self.order_service.query_user_orders(self.user_id, limit=5)
        
        if not orders:
            return "您还没有订单记录"
        
        result = "您的最近订单：\n\n"
        for i, order in enumerate(orders, 1):
            result += f"{i}. 订单号：{order['order_id']}\n"
            result += f"   状态：{order['status']}\n"
            result += f"   金额：￥{order['total']}\n"
            result += f"   时间：{order['created_at']}\n\n"
        
        return result
    
    def _cancel_order_tool(self, order_id: str) -> str:
        """取消订单工具"""
        result = self.order_service.cancel_order(order_id.strip())
        
        if result["success"]:
            return f"{result['message']}\n退款金额：￥{result['refund_amount']}\n退款将在3-5个工作日内原路返回"
        else:
            return f"取消失败：{result['message']}"
    
    def _track_shipping_tool(self, tracking_no: str) -> str:
        """物流追踪工具"""
        tracking = self.order_service.track_shipping(tracking_no.strip())
        
        if "error" in tracking:
            return tracking["error"]
        
        result = f"""物流信息：
物流公司：{tracking['carrier']}
当前状态：{tracking['status']}
当前位置：{tracking['current_location']}
预计送达：{tracking['estimated_delivery']}

物流轨迹：
"""
        for record in tracking['history']:
            result += f"{record['time']} - {record['status']} - {record['location']}\n"
        
        return result
    
    def _search_faq_tool(self, query: str) -> str:
        """搜索FAQ工具"""
        faqs = self.faq_service.search_faq(query)
        
        if not faqs:
            return "未找到相关问题，建议您：\n1. 换个关键词重新搜索\n2. 转接人工客服获取帮助"
        
        result = "为您找到以下相关问题：\n\n"
        for i, faq in enumerate(faqs, 1):
            result += f"{i}. {faq['question']}\n"
            result += f"{faq['answer']}\n"
            result += f"（相关度：{faq['relevance_score']}）\n\n"
        
        return result
    
    def _search_products_tool(self, keyword: str) -> str:
        """搜索商品工具"""
        products = self.product_service.search_products(keyword)
        
        if not products:
            return f"未找到与 '{keyword}' 相关的商品"
        
        result = f"为您找到 {len(products)} 个相关商品：\n\n"
        for product in products:
            result += f"【{product['name']}】\n"
            result += f"价格：￥{product['price']}\n"
            result += f"库存：{product['stock']} 件\n"
            result += f"描述：{product['description']}\n"
            result += f"商品ID：{product['id']}\n\n"
        
        return result
    
    def _check_stock_tool(self, product_id: str) -> str:
        """检查库存工具"""
        stock_info = self.product_service.check_stock(product_id.strip())
        return stock_info["message"]
    
    def _create_agent(self):
        """创建客服 Agent"""
        prompt = ChatPromptTemplate.from_messages([
            ("system", """你是一个专业、友好的电商客服助手。

你的职责：
1. 热情、耐心地帮助用户解决问题
2. 使用工具查询订单、物流、商品等信息
3. 回答用户关于退换货、发票、配送等问题
4. 在无法解决时，引导用户转接人工客服

沟通规范：
- 使用礼貌、专业的语言
- 回答要准确、完整
- 主动询问是否还有其他问题
- 对于投诉要表示理解和歉意
- 保护用户隐私，不泄露敏感信息

特殊情况处理：
- 如果用户情绪激动，先安抚情绪
- 如果问题复杂，建议转人工
- 如果涉及退款，说明清楚流程和时间

当前用户ID：{user_id}
"""),
            MessagesPlaceholder(variable_name="chat_history"),
            ("human", "{input}"),
            MessagesPlaceholder(variable_name="agent_scratchpad")
        ])
        
        agent = create_openai_functions_agent(
            llm=self.llm,
            tools=self.tools,
            prompt=prompt
        )
        
        return AgentExecutor(
            agent=agent,
            tools=self.tools,
            memory=self.memory,
            verbose=True,
            max_iterations=5,
            handle_parsing_errors=True
        )
    
    def chat(self, user_input: str) -> Dict[str, Any]:
        """处理用户输入"""
        
        # 1. 意图识别
        intent_result = self.intent_classifier.classify(user_input)
        logger.info(f"识别意图: {intent_result['intent']}")
        
        # 2. 特殊意图处理
        if intent_result['intent'] == 'human' or intent_result['confidence'] < 0.3:
            return {
                "response": "好的，正在为您转接人工客服，请稍候...",
                "intent": "human_transfer",
                "need_human": True
            }
        
        # 3. 执行 Agent
        try:
            result = self.agent.invoke({
                "input": user_input,
                "user_id": self.user_id
            })
            
            response_text = result["output"]
            
            # 4. 添加满意度调查（每5轮对话）
            if len(self.memory.chat_memory.messages) % 10 == 0:
                response_text += "\n\n【满意度调查】请问本次服务您满意吗？（满意/一般/不满意）"
            
            return {
                "response": response_text,
                "intent": intent_result['intent'],
                "confidence": intent_result['confidence'],
                "need_human": False
            }
            
        except Exception as e:
            logger.error(f"Agent 执行失败: {str(e)}")
            return {
                "response": "抱歉，系统出现了一点问题。正在为您转接人工客服...",
                "intent": "error",
                "need_human": True,
                "error": str(e)
            }
    
    def get_conversation_history(self) -> List[Dict[str, str]]:
        """获取对话历史"""
        messages = self.memory.chat_memory.messages
        history = []
        
        for msg in messages:
            history.append({
                "role": "user" if msg.type == "human" else "assistant",
                "content": msg.content
            })
        
        return history
    
    def clear_history(self):
        """清空对话历史"""
        self.memory.clear()
        self.session_state = {
            "current_order": None,
            "pending_action": None,
            "satisfaction_score": None
        }


# 使用示例
def demo_customer_service():
    """演示客服系统"""
    
    # 创建客服 Agent
    agent = CustomerServiceAgent(user_id="USER001")
    
    print("=" * 60)
    print("欢迎使用智能客服系统")
    print("=" * 60)
    
    # 模拟对话场景
    conversations = [
        "你好，我想查询我的订单",
        "订单号是 ORD20240301001",
        "这个订单什么时候能到？",
        "我想取消订单 ORD20240302001",
        "退款多久能到账？",
        "谢谢"
    ]
    
    for user_input in conversations:
        print(f"\n用户: {user_input}")
        
        result = agent.chat(user_input)
        
        print(f"客服: {result['response']}")
        print(f"[意图: {result['intent']}, 置信度: {result.get('confidence', 'N/A')}]")
        
        if result.get('need_human'):
            print("[系统提示: 需要转接人工客服]")
            break
        
        print("-" * 60)


if __name__ == "__main__":
    demo_customer_service()
```

### 1.4 核心难点与解决方案

#### 难点 1：意图识别准确率

**问题**：
- 用户表达方式多样，同一意图有多种说法
- 多意图混合（如：查询订单 + 投诉）
- 口语化、错别字、方言

**解决方案**：
```python
class EnhancedIntentClassifier:
    """增强的意图识别器"""
    
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
        
        # 使用 Few-Shot Learning
        self.examples = {
            "order": [
                "我的订单到哪了",
                "帮我查下快递",
                "订单啥时候发货",
                "ORD123456 这个单子咋样了"
            ],
            "refund": [
                "我要退货",
                "这个东西不想要了",
                "能退款吗",
                "申请退货退款"
            ]
        }
        
        # 使用向量相似度辅助
        self.embeddings = OpenAIEmbeddings()
        self._build_intent_vectors()
    
    def _build_intent_vectors(self):
        """构建意图向量库"""
        texts = []
        metadatas = []
        
        for intent, examples in self.examples.items():
            for example in examples:
                texts.append(example)
                metadatas.append({"intent": intent})
        
        self.intent_vectorstore = Chroma.from_texts(
            texts=texts,
            embedding=self.embeddings,
            metadatas=metadatas,
            collection_name="intent_examples"
        )
    
    def classify_hybrid(self, user_input: str) -> Dict[str, Any]:
        """混合方法识别意图"""
        
        # 方法1：向量相似度
        vector_results = self.intent_vectorstore.similarity_search_with_score(
            user_input, k=3
        )
        vector_intent = vector_results[0][0].metadata["intent"] if vector_results else None
        vector_score = 1 - vector_results[0][1] if vector_results else 0
        
        # 方法2：LLM 分类
        llm_result = self._llm_classify(user_input)
        
        # 方法3：规则匹配
        rule_result = self._rule_based_classify(user_input)
        
        # 综合判断
        final_intent = self._ensemble_decision(
            vector_intent, vector_score,
            llm_result,
            rule_result
        )
        
        return final_intent
    
    def _rule_based_classify(self, text: str) -> Dict[str, Any]:
        """基于规则的分类"""
        rules = {
            "order": ["订单", "快递", "物流", "发货"],
            "refund": ["退货", "退款", "不想要"],
            "complaint": ["投诉", "差评", "质量问题"]
        }
        
        for intent, keywords in rules.items():
            if any(kw in text for kw in keywords):
                return {"intent": intent, "confidence": 0.8}
        
        return {"intent": "unknown", "confidence": 0.0}
```

#### 难点 2：上下文理解

**问题**：
- 用户使用代词（"它"、"这个"）
- 省略主语
- 跨轮次的信息关联

**解决方案**：
```python
class ContextAwareAgent:
    """上下文感知 Agent"""
    
    def __init__(self):
        self.context_tracker = {
            "mentioned_orders": [],
            "mentioned_products": [],
            "last_action": None,
            "unresolved_issues": []
        }
    
    def process_with_context(self, user_input: str) -> str:
        """带上下文处理"""
        
        # 解析指代
        resolved_input = self._resolve_references(user_input)
        
        # 补全省略信息
        complete_input = self._complete_ellipsis(resolved_input)
        
        # 执行
        response = self.agent.invoke({"input": complete_input})
        
        # 更新上下文
        self._update_context(user_input, response)
        
        return response
    
    def _resolve_references(self, text: str) -> str:
        """解析指代"""
        # 如果提到"它"、"这个订单"等
        if any(word in text for word in ["它", "这个", "那个"]):
            # 查找最近提到的实体
            if self.context_tracker["mentioned_orders"]:
                last_order = self.context_tracker["mentioned_orders"][-1]
                text = text.replace("它", last_order)
                text = text.replace("这个订单", last_order)
        
        return text
```



#### 难点 3：高并发处理

**问题**：
- 日均10万+咨询，峰值QPS可达1000+
- LLM API 调用延迟高（2-5秒）
- 成本控制

**解决方案**：
```python
import asyncio
from typing import List
import aioredis
from functools import lru_cache

class HighPerformanceCustomerService:
    """高性能客服系统"""
    
    def __init__(self):
        # 使用连接池
        self.llm_pool = self._create_llm_pool(pool_size=10)
        
        # Redis 缓存
        self.redis = aioredis.from_url("redis://localhost")
        
        # 请求队列
        self.request_queue = asyncio.Queue(maxsize=1000)
        
        # 启动工作线程
        self.workers = [
            asyncio.create_task(self._worker())
            for _ in range(10)
        ]
    
    async def _worker(self):
        """工作线程"""
        while True:
            request = await self.request_queue.get()
            
            try:
                # 检查缓存
                cached = await self._get_cache(request['input'])
                if cached:
                    request['callback'](cached)
                    continue
                
                # 处理请求
                response = await self._process_request(request)
                
                # 保存缓存
                await self._set_cache(request['input'], response)
                
                request['callback'](response)
                
            except Exception as e:
                logger.error(f"处理失败: {str(e)}")
                request['callback']({"error": str(e)})
            
            finally:
                self.request_queue.task_done()
    
    async def _get_cache(self, key: str) -> Optional[str]:
        """获取缓存"""
        cache_key = f"cs:response:{hash(key)}"
        return await self.redis.get(cache_key)
    
    async def _set_cache(self, key: str, value: str, ttl: int = 3600):
        """设置缓存"""
        cache_key = f"cs:response:{hash(key)}"
        await self.redis.setex(cache_key, ttl, value)
    
    @lru_cache(maxsize=1000)
    def _get_faq_answer(self, question: str) -> Optional[str]:
        """FAQ 内存缓存"""
        # 常见问题直接返回，不调用 LLM
        pass
    
    async def handle_request(self, user_input: str) -> str:
        """处理请求（异步）"""
        
        # 1. 快速路径：FAQ 直接返回
        faq_answer = self._get_faq_answer(user_input)
        if faq_answer:
            return faq_answer
        
        # 2. 加入队列
        future = asyncio.Future()
        
        await self.request_queue.put({
            'input': user_input,
            'callback': future.set_result
        })
        
        # 3. 等待结果
        return await future


# 性能优化配置
OPTIMIZATION_CONFIG = {
    "cache": {
        "enabled": True,
        "ttl": 3600,  # 1小时
        "max_size": 10000
    },
    "rate_limit": {
        "max_requests_per_second": 100,
        "max_requests_per_user": 10
    },
    "timeout": {
        "llm_call": 10,  # 10秒超时
        "total_request": 30  # 总超时30秒
    },
    "fallback": {
        "use_simple_model": True,  # 降级使用简单模型
        "simple_model": "gpt-3.5-turbo"
    }
}
```

#### 难点 4：多轮对话状态管理

**问题**：
- 订单确认需要多轮交互
- 用户中途切换话题
- 状态丢失导致重复询问

**解决方案**：
```python
from enum import Enum
from typing import Optional, Dict, Any

class ConversationState(Enum):
    """对话状态"""
    IDLE = "idle"
    QUERYING_ORDER = "querying_order"
    CONFIRMING_CANCEL = "confirming_cancel"
    COLLECTING_FEEDBACK = "collecting_feedback"
    WAITING_HUMAN = "waiting_human"


class StatefulCustomerService:
    """带状态管理的客服系统"""
    
    def __init__(self, user_id: str):
        self.user_id = user_id
        self.state = ConversationState.IDLE
        
        # 状态数据
        self.state_data = {
            "pending_order_id": None,
            "cancel_reason": None,
            "feedback_score": None,
            "context": {}
        }
        
        # 状态转换规则
        self.state_transitions = {
            ConversationState.IDLE: {
                "cancel_order": ConversationState.CONFIRMING_CANCEL,
                "query_order": ConversationState.QUERYING_ORDER
            },
            ConversationState.CONFIRMING_CANCEL: {
                "confirm": ConversationState.IDLE,
                "cancel": ConversationState.IDLE
            }
        }
    
    def process_with_state(self, user_input: str) -> Dict[str, Any]:
        """带状态处理"""
        
        # 根据当前状态处理
        if self.state == ConversationState.CONFIRMING_CANCEL:
            return self._handle_cancel_confirmation(user_input)
        
        elif self.state == ConversationState.COLLECTING_FEEDBACK:
            return self._handle_feedback(user_input)
        
        else:
            # 正常处理
            return self._handle_normal(user_input)
    
    def _handle_cancel_confirmation(self, user_input: str) -> Dict[str, Any]:
        """处理取消确认"""
        
        if any(word in user_input for word in ["确认", "是的", "对", "取消"]):
            # 执行取消
            order_id = self.state_data["pending_order_id"]
            result = self.order_service.cancel_order(order_id)
            
            # 重置状态
            self.state = ConversationState.IDLE
            self.state_data["pending_order_id"] = None
            
            return {
                "response": f"订单已取消。{result['message']}",
                "state": self.state.value
            }
        
        elif any(word in user_input for word in ["不", "算了", "不取消"]):
            # 取消操作
            self.state = ConversationState.IDLE
            self.state_data["pending_order_id"] = None
            
            return {
                "response": "好的，已取消操作。还有什么可以帮您的吗？",
                "state": self.state.value
            }
        
        else:
            # 继续确认
            return {
                "response": "请明确回复：确认取消 或 不取消",
                "state": self.state.value
            }
    
    def transition_state(self, action: str):
        """状态转换"""
        if self.state in self.state_transitions:
            if action in self.state_transitions[self.state]:
                new_state = self.state_transitions[self.state][action]
                logger.info(f"状态转换: {self.state.value} -> {new_state.value}")
                self.state = new_state
```

### 1.5 生产环境部署

#### 部署架构

```
                    ┌─────────────┐
                    │   Nginx     │
                    │  (负载均衡)  │
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
   │ Agent   │       │ Agent   │       │ Agent   │
   │ Server1 │       │ Server2 │       │ Server3 │
   └────┬────┘       └────┬────┘       └────┬────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
   │  Redis  │       │  MySQL  │       │  ES     │
   │ (缓存)  │       │ (数据)  │       │ (日志)  │
   └─────────┘       └─────────┘       └─────────┘
```

#### 部署配置

```python
# config/production.py

PRODUCTION_CONFIG = {
    "llm": {
        "provider": "openai",
        "model": "gpt-4",
        "temperature": 0.7,
        "max_tokens": 1000,
        "timeout": 30,
        "retry": {
            "max_attempts": 3,
            "backoff_factor": 2
        }
    },
    
    "cache": {
        "redis": {
            "host": "redis-cluster.internal",
            "port": 6379,
            "db": 0,
            "password": "${REDIS_PASSWORD}",
            "max_connections": 50
        },
        "ttl": {
            "faq": 86400,  # 24小时
            "order": 300,  # 5分钟
            "product": 3600  # 1小时
        }
    },
    
    "database": {
        "mysql": {
            "host": "mysql-master.internal",
            "port": 3306,
            "database": "customer_service",
            "user": "cs_app",
            "password": "${MYSQL_PASSWORD}",
            "pool_size": 20
        }
    },
    
    "monitoring": {
        "prometheus": {
            "enabled": True,
            "port": 9090
        },
        "logging": {
            "level": "INFO",
            "format": "json",
            "output": "elasticsearch"
        }
    },
    
    "rate_limit": {
        "global": 1000,  # 全局QPS
        "per_user": 10   # 单用户QPS
    },
    
    "circuit_breaker": {
        "failure_threshold": 5,
        "timeout": 60,
        "half_open_timeout": 30
    }
}
```

#### 监控指标

```python
from prometheus_client import Counter, Histogram, Gauge
import time

# 定义监控指标
request_count = Counter(
    'cs_requests_total',
    'Total customer service requests',
    ['intent', 'status']
)

request_duration = Histogram(
    'cs_request_duration_seconds',
    'Request duration in seconds',
    ['intent']
)

active_sessions = Gauge(
    'cs_active_sessions',
    'Number of active customer service sessions'
)

llm_call_count = Counter(
    'cs_llm_calls_total',
    'Total LLM API calls',
    ['model', 'status']
)


class MonitoredCustomerService:
    """带监控的客服系统"""
    
    def chat(self, user_input: str) -> Dict[str, Any]:
        """处理请求（带监控）"""
        
        start_time = time.time()
        
        try:
            # 增加活跃会话数
            active_sessions.inc()
            
            # 识别意图
            intent_result = self.intent_classifier.classify(user_input)
            intent = intent_result['intent']
            
            # 处理请求
            response = self.agent.invoke({"input": user_input})
            
            # 记录成功
            request_count.labels(intent=intent, status='success').inc()
            
            return response
            
        except Exception as e:
            # 记录失败
            request_count.labels(intent='unknown', status='error').inc()
            logger.error(f"请求失败: {str(e)}")
            raise
            
        finally:
            # 记录耗时
            duration = time.time() - start_time
            request_duration.labels(intent=intent).observe(duration)
            
            # 减少活跃会话数
            active_sessions.dec()
```

### 1.6 效果评估

#### 评估指标

```python
class CustomerServiceMetrics:
    """客服系统评估指标"""
    
    def __init__(self):
        self.metrics = {
            "response_time": [],      # 响应时间
            "resolution_rate": 0,     # 问题解决率
            "satisfaction_score": 0,  # 满意度
            "human_transfer_rate": 0, # 转人工率
            "cost_per_session": 0     # 单次会话成本
        }
    
    def calculate_metrics(self, sessions: List[Dict]) -> Dict[str, float]:
        """计算指标"""
        
        total_sessions = len(sessions)
        resolved_sessions = 0
        total_satisfaction = 0
        human_transfers = 0
        total_cost = 0
        
        for session in sessions:
            # 响应时间
            self.metrics["response_time"].append(session["duration"])
            
            # 问题解决
            if session["resolved"]:
                resolved_sessions += 1
            
            # 满意度
            if session.get("satisfaction"):
                total_satisfaction += session["satisfaction"]
            
            # 转人工
            if session.get("human_transfer"):
                human_transfers += 1
            
            # 成本（LLM调用次数 * 单价）
            total_cost += session["llm_calls"] * 0.002  # $0.002/call
        
        return {
            "avg_response_time": sum(self.metrics["response_time"]) / total_sessions,
            "resolution_rate": resolved_sessions / total_sessions,
            "satisfaction_score": total_satisfaction / total_sessions,
            "human_transfer_rate": human_transfers / total_sessions,
            "cost_per_session": total_cost / total_sessions
        }


# 实际效果数据（基于真实案例）
ACTUAL_METRICS = {
    "before_ai": {
        "avg_response_time": 180,  # 3分钟
        "resolution_rate": 0.65,
        "satisfaction_score": 3.8,
        "cost_per_session": 5.0,  # 人工成本
        "daily_capacity": 1000
    },
    "after_ai": {
        "avg_response_time": 15,   # 15秒
        "resolution_rate": 0.82,
        "satisfaction_score": 4.2,
        "cost_per_session": 0.05,  # AI成本
        "daily_capacity": 50000,
        "human_transfer_rate": 0.15  # 15%转人工
    },
    "improvement": {
        "response_time": "-92%",
        "resolution_rate": "+26%",
        "satisfaction": "+10.5%",
        "cost": "-99%",
        "capacity": "+4900%"
    }
}
```

### 1.7 常见问题与解决方案

#### 问题 1：用户不信任 AI

**现象**：用户一开始就要求转人工

**解决方案**：
```python
def build_trust_strategy():
    """建立信任策略"""
    
    strategies = {
        "透明化": "明确告知用户正在与AI对话",
        "能力展示": "快速准确回答第一个问题",
        "人工保底": "随时可以转人工，降低用户顾虑",
        "个性化": "记住用户偏好，提供个性化服务"
    }
    
    # 开场白示例
    greeting = """您好！我是智能客服助手小智。
    
我可以帮您：
✓ 查询订单和物流
✓ 处理退换货
✓ 解答常见问题

如需人工服务，随时告诉我。现在有什么可以帮您的吗？"""
    
    return greeting
```

#### 问题 2：理解偏差导致错误回答

**现象**：用户问A，AI答B

**解决方案**：
```python
class ConfidenceBasedResponse:
    """基于置信度的响应策略"""
    
    def respond_with_confidence(self, query: str, confidence: float) -> str:
        """根据置信度决定响应策略"""
        
        if confidence > 0.8:
            # 高置信度：直接回答
            return self.generate_answer(query)
        
        elif confidence > 0.5:
            # 中等置信度：确认后回答
            return f"您是想问关于{self.extract_topic(query)}的问题吗？"
        
        else:
            # 低置信度：请求澄清
            return "抱歉，我没有完全理解您的问题。您能详细说明一下吗？"
```

---

## 案例 2：企业知识库问答系统

### 2.1 业务场景

某大型企业有海量内部文档（技术文档、规章制度、操作手册等），员工查找信息效率低。需要构建智能问答系统：
- 支持自然语言查询
- 多文档类型（PDF、Word、Wiki、代码）
- 精准定位答案来源
- 支持多轮追问

### 2.2 系统架构

```
文档源
  ├─ Confluence Wiki
  ├─ GitLab 代码仓库
  ├─ 共享文件夹
  └─ 数据库文档

        ↓ 文档采集

文档处理层
  ├─ 文本提取
  ├─ 分块切分
  ├─ 向量化
  └─ 索引构建

        ↓

向量数据库
  (Chroma/Pinecone)

        ↓

RAG Agent
  ├─ 查询理解
  ├─ 检索增强
  ├─ 答案生成
  └─ 来源标注

        ↓

用户界面
```

### 2.3 核心实现

```python
from langchain.document_loaders import (
    PyPDFLoader,
    UnstructuredWordDocumentLoader,
    GitLoader,
    ConfluenceLoader
)
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain.chains import RetrievalQA
from langchain.prompts import PromptTemplate
from typing import List, Dict, Any
import os
from pathlib import Path

class DocumentProcessor:
    """文档处理器"""
    
    def __init__(self):
        self.text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=1000,
            chunk_overlap=200,
            length_function=len,
            separators=["\n\n", "\n", "。", "！", "？", "；", " ", ""]
        )
    
    def process_pdf(self, file_path: str) -> List[Dict]:
        """处理 PDF 文档"""
        loader = PyPDFLoader(file_path)
        documents = loader.load()
        
        # 分块
        chunks = self.text_splitter.split_documents(documents)
        
        # 添加元数据
        for chunk in chunks:
            chunk.metadata.update({
                "source_type": "pdf",
                "file_name": Path(file_path).name,
                "file_path": file_path
            })
        
        return chunks
    
    def process_word(self, file_path: str) -> List[Dict]:
        """处理 Word 文档"""
        loader = UnstructuredWordDocumentLoader(file_path)
        documents = loader.load()
        
        chunks = self.text_splitter.split_documents(documents)
        
        for chunk in chunks:
            chunk.metadata.update({
                "source_type": "word",
                "file_name": Path(file_path).name
            })
        
        return chunks
    
    def process_code_repo(self, repo_path: str, file_extensions: List[str]) -> List[Dict]:
        """处理代码仓库"""
        loader = GitLoader(
            repo_path=repo_path,
            file_filter=lambda file_path: any(
                file_path.endswith(ext) for ext in file_extensions
            )
        )
        documents = loader.load()
        
        # 代码使用不同的分块策略
        code_splitter = RecursiveCharacterTextSplitter(
            chunk_size=500,
            chunk_overlap=50,
            separators=["\n\nclass ", "\n\ndef ", "\n\n", "\n", " "]
        )
        
        chunks = code_splitter.split_documents(documents)
        
        for chunk in chunks:
            chunk.metadata.update({
                "source_type": "code",
                "repo_path": repo_path
            })
        
        return chunks
    
    def process_confluence(self, space_key: str, confluence_url: str) -> List[Dict]:
        """处理 Confluence Wiki"""
        loader = ConfluenceLoader(
            url=confluence_url,
            space_key=space_key
        )
        documents = loader.load()
        
        chunks = self.text_splitter.split_documents(documents)
        
        for chunk in chunks:
            chunk.metadata.update({
                "source_type": "confluence",
                "space": space_key
            })
        
        return chunks


class KnowledgeBaseBuilder:
    """知识库构建器"""
    
    def __init__(self, persist_directory: str = "./chroma_db"):
        self.persist_directory = persist_directory
        self.embeddings = OpenAIEmbeddings()
        self.processor = DocumentProcessor()
        self.vectorstore = None
    
    def build_from_sources(self, sources: Dict[str, Any]):
        """从多个数据源构建知识库"""
        
        all_chunks = []
        
        # 处理 PDF 文件
        if "pdf_files" in sources:
            for pdf_file in sources["pdf_files"]:
                logger.info(f"处理 PDF: {pdf_file}")
                chunks = self.processor.process_pdf(pdf_file)
                all_chunks.extend(chunks)
        
        # 处理 Word 文件
        if "word_files" in sources:
            for word_file in sources["word_files"]:
                logger.info(f"处理 Word: {word_file}")
                chunks = self.processor.process_word(word_file)
                all_chunks.extend(chunks)
        
        # 处理代码仓库
        if "code_repos" in sources:
            for repo in sources["code_repos"]:
                logger.info(f"处理代码仓库: {repo['path']}")
                chunks = self.processor.process_code_repo(
                    repo["path"],
                    repo.get("extensions", [".py", ".js", ".java"])
                )
                all_chunks.extend(chunks)
        
        # 处理 Confluence
        if "confluence" in sources:
            for space in sources["confluence"]["spaces"]:
                logger.info(f"处理 Confluence 空间: {space}")
                chunks = self.processor.process_confluence(
                    space,
                    sources["confluence"]["url"]
                )
                all_chunks.extend(chunks)
        
        logger.info(f"总共处理了 {len(all_chunks)} 个文档块")
        
        # 构建向量库
        self.vectorstore = Chroma.from_documents(
            documents=all_chunks,
            embedding=self.embeddings,
            persist_directory=self.persist_directory
        )
        
        self.vectorstore.persist()
        logger.info("知识库构建完成")
    
    def update_document(self, file_path: str):
        """更新单个文档"""
        # 删除旧文档
        self.vectorstore.delete(
            filter={"file_path": file_path}
        )
        
        # 添加新文档
        if file_path.endswith(".pdf"):
            chunks = self.processor.process_pdf(file_path)
        elif file_path.endswith(".docx"):
            chunks = self.processor.process_word(file_path)
        else:
            raise ValueError(f"不支持的文件类型: {file_path}")
        
        self.vectorstore.add_documents(chunks)
        self.vectorstore.persist()


class EnterpriseQAAgent:
    """企业知识库问答 Agent"""
    
    def __init__(self, vectorstore: Chroma):
        self.vectorstore = vectorstore
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
        
        # 创建检索器
        self.retriever = vectorstore.as_retriever(
            search_type="mmr",  # 最大边际相关性
            search_kwargs={
                "k": 5,  # 返回5个最相关的文档
                "fetch_k": 20,  # 先获取20个候选
                "lambda_mult": 0.5  # 多样性参数
            }
        )
        
        # 创建 QA 链
        self.qa_chain = self._create_qa_chain()
    
    def _create_qa_chain(self):
        """创建问答链"""
        
        prompt_template = """你是一个企业知识库助手。请基于以下文档内容回答问题。

要求：
1. 答案必须基于提供的文档内容
2. 如果文档中没有相关信息，明确说明"文档中未找到相关信息"
3. 引用具体的文档来源
4. 如果有多个相关答案，都要列出
5. 使用清晰的格式组织答案

文档内容：
{context}

问题：{question}

回答："""
        
        PROMPT = PromptTemplate(
            template=prompt_template,
            input_variables=["context", "question"]
        )
        
        return RetrievalQA.from_chain_type(
            llm=self.llm,
            chain_type="stuff",
            retriever=self.retriever,
            return_source_documents=True,
            chain_type_kwargs={"prompt": PROMPT}
        )
    
    def query(self, question: str) -> Dict[str, Any]:
        """查询知识库"""
        
        # 执行查询
        result = self.qa_chain({"query": question})
        
        # 格式化响应
        response = {
            "answer": result["result"],
            "sources": self._format_sources(result["source_documents"]),
            "confidence": self._calculate_confidence(result["source_documents"])
        }
        
        return response
    
    def _format_sources(self, source_docs: List) -> List[Dict]:
        """格式化来源信息"""
        sources = []
        
        for doc in source_docs:
            source_info = {
                "content": doc.page_content[:200] + "...",  # 摘要
                "metadata": doc.metadata
            }
            sources.append(source_info)
        
        return sources
    
    def _calculate_confidence(self, source_docs: List) -> float:
        """计算置信度"""
        if not source_docs:
            return 0.0
        
        # 简单策略：基于检索到的文档数量和相关性
        # 实际应该使用更复杂的算法
        return min(len(source_docs) / 5.0, 1.0)


# 使用示例
def demo_knowledge_base():
    """演示知识库系统"""
    
    # 1. 构建知识库
    builder = KnowledgeBaseBuilder()
    
    sources = {
        "pdf_files": [
            "./docs/employee_handbook.pdf",
            "./docs/technical_guide.pdf"
        ],
        "word_files": [
            "./docs/process_manual.docx"
        ],
        "code_repos": [
            {
                "path": "./repos/backend",
                "extensions": [".py", ".md"]
            }
        ],
        "confluence": {
            "url": "https://wiki.company.com",
            "spaces": ["ENG", "HR", "OPS"]
        }
    }
    
    builder.build_from_sources(sources)
    
    # 2. 创建问答 Agent
    agent = EnterpriseQAAgent(builder.vectorstore)
    
    # 3. 查询示例
    questions = [
        "公司的年假政策是什么？",
        "如何部署后端服务？",
        "代码审查的流程是怎样的？"
    ]
    
    for question in questions:
        print(f"\n问题: {question}")
        result = agent.query(question)
        print(f"回答: {result['answer']}")
        print(f"置信度: {result['confidence']:.2f}")
        print(f"来源数量: {len(result['sources'])}")
```



### 2.4 核心难点与解决方案

#### 难点 1：文档更新同步

**问题**：
- 文档频繁更新，向量库需要实时同步
- 全量重建耗时长（数万文档需要数小时）
- 增量更新如何保证一致性

**解决方案**：
```python
from datetime import datetime
from typing import Set
import hashlib

class IncrementalIndexer:
    """增量索引器"""
    
    def __init__(self, vectorstore: Chroma):
        self.vectorstore = vectorstore
        self.document_registry = {}  # 文档注册表
        self._load_registry()
    
    def _load_registry(self):
        """加载文档注册表"""
        # 从数据库加载已索引文档的元信息
        # 包括：文件路径、最后修改时间、内容哈希
        pass
    
    def sync_documents(self, source_paths: List[str]) -> Dict[str, Any]:
        """同步文档"""
        
        stats = {
            "added": 0,
            "updated": 0,
            "deleted": 0,
            "unchanged": 0
        }
        
        current_files = set()
        
        for source_path in source_paths:
            for file_path in self._scan_files(source_path):
                current_files.add(file_path)
                
                # 计算文件哈希
                file_hash = self._calculate_file_hash(file_path)
                file_mtime = os.path.getmtime(file_path)
                
                # 检查是否需要更新
                if file_path not in self.document_registry:
                    # 新文档
                    self._add_document(file_path, file_hash, file_mtime)
                    stats["added"] += 1
                    
                elif self.document_registry[file_path]["hash"] != file_hash:
                    # 文档已修改
                    self._update_document(file_path, file_hash, file_mtime)
                    stats["updated"] += 1
                    
                else:
                    # 文档未变化
                    stats["unchanged"] += 1
        
        # 删除已不存在的文档
        registered_files = set(self.document_registry.keys())
        deleted_files = registered_files - current_files
        
        for file_path in deleted_files:
            self._delete_document(file_path)
            stats["deleted"] += 1
        
        logger.info(f"同步完成: {stats}")
        return stats
    
    def _calculate_file_hash(self, file_path: str) -> str:
        """计算文件哈希"""
        hasher = hashlib.md5()
        with open(file_path, 'rb') as f:
            for chunk in iter(lambda: f.read(4096), b""):
                hasher.update(chunk)
        return hasher.hexdigest()
    
    def _add_document(self, file_path: str, file_hash: str, mtime: float):
        """添加文档到向量库"""
        # 处理文档
        chunks = self.processor.process_document(file_path)
        
        # 添加到向量库
        self.vectorstore.add_documents(chunks)
        
        # 更新注册表
        self.document_registry[file_path] = {
            "hash": file_hash,
            "mtime": mtime,
            "indexed_at": datetime.now().isoformat()
        }
    
    def _update_document(self, file_path: str, file_hash: str, mtime: float):
        """更新文档"""
        # 先删除旧版本
        self.vectorstore.delete(
            filter={"file_path": file_path}
        )
        
        # 添加新版本
        self._add_document(file_path, file_hash, mtime)
    
    def _delete_document(self, file_path: str):
        """删除文档"""
        self.vectorstore.delete(
            filter={"file_path": file_path}
        )
        
        del self.document_registry[file_path]


# 定时同步任务
class DocumentSyncScheduler:
    """文档同步调度器"""
    
    def __init__(self, indexer: IncrementalIndexer):
        self.indexer = indexer
        self.running = False
    
    def start(self, interval_minutes: int = 30):
        """启动定时同步"""
        self.running = True
        
        async def sync_loop():
            while self.running:
                try:
                    logger.info("开始同步文档...")
                    stats = self.indexer.sync_documents([
                        "./docs",
                        "./repos"
                    ])
                    logger.info(f"同步完成: {stats}")
                    
                except Exception as e:
                    logger.error(f"同步失败: {str(e)}")
                
                # 等待下次同步
                await asyncio.sleep(interval_minutes * 60)
        
        asyncio.create_task(sync_loop())
```

#### 难点 2：跨语言代码理解

**问题**：
- 代码仓库包含多种语言（Python、Java、Go、JavaScript）
- 不同语言的语法和结构差异大
- 需要理解代码逻辑而非仅仅文本匹配

**解决方案**：
```python
from tree_sitter import Language, Parser
import tree_sitter_python
import tree_sitter_java
import tree_sitter_javascript

class CodeAnalyzer:
    """代码分析器"""
    
    def __init__(self):
        # 初始化多语言解析器
        self.parsers = {
            "python": self._create_parser(tree_sitter_python.language()),
            "java": self._create_parser(tree_sitter_java.language()),
            "javascript": self._create_parser(tree_sitter_javascript.language())
        }
    
    def _create_parser(self, language) -> Parser:
        """创建解析器"""
        parser = Parser()
        parser.set_language(language)
        return parser
    
    def extract_code_structure(self, code: str, language: str) -> Dict[str, Any]:
        """提取代码结构"""
        
        if language not in self.parsers:
            return {"error": f"不支持的语言: {language}"}
        
        parser = self.parsers[language]
        tree = parser.parse(bytes(code, "utf8"))
        
        structure = {
            "classes": [],
            "functions": [],
            "imports": [],
            "comments": []
        }
        
        # 遍历语法树
        self._traverse_tree(tree.root_node, structure, language)
        
        return structure
    
    def _traverse_tree(self, node, structure: Dict, language: str):
        """遍历语法树"""
        
        if language == "python":
            if node.type == "class_definition":
                structure["classes"].append({
                    "name": self._get_node_text(node, "name"),
                    "line": node.start_point[0],
                    "docstring": self._extract_docstring(node)
                })
            
            elif node.type == "function_definition":
                structure["functions"].append({
                    "name": self._get_node_text(node, "name"),
                    "line": node.start_point[0],
                    "parameters": self._extract_parameters(node),
                    "docstring": self._extract_docstring(node)
                })
        
        # 递归处理子节点
        for child in node.children:
            self._traverse_tree(child, structure, language)
    
    def generate_code_summary(self, code: str, language: str) -> str:
        """生成代码摘要"""
        
        structure = self.extract_code_structure(code, language)
        
        summary_parts = []
        
        # 类摘要
        if structure["classes"]:
            summary_parts.append(f"包含 {len(structure['classes'])} 个类:")
            for cls in structure["classes"]:
                summary_parts.append(f"  - {cls['name']}: {cls.get('docstring', '无描述')}")
        
        # 函数摘要
        if structure["functions"]:
            summary_parts.append(f"\n包含 {len(structure['functions'])} 个函数:")
            for func in structure["functions"]:
                params = ", ".join(func.get("parameters", []))
                summary_parts.append(f"  - {func['name']}({params}): {func.get('docstring', '无描述')}")
        
        return "\n".join(summary_parts)


class EnhancedCodeQAAgent:
    """增强的代码问答 Agent"""
    
    def __init__(self, vectorstore: Chroma):
        self.vectorstore = vectorstore
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
        self.code_analyzer = CodeAnalyzer()
    
    def query_code(self, question: str) -> Dict[str, Any]:
        """查询代码相关问题"""
        
        # 1. 检索相关代码
        docs = self.vectorstore.similarity_search(question, k=5)
        
        # 2. 分析代码结构
        code_contexts = []
        for doc in docs:
            if doc.metadata.get("source_type") == "code":
                # 提取代码结构
                language = self._detect_language(doc.metadata.get("file_path", ""))
                structure = self.code_analyzer.extract_code_structure(
                    doc.page_content,
                    language
                )
                
                code_contexts.append({
                    "file": doc.metadata.get("file_path"),
                    "content": doc.page_content,
                    "structure": structure,
                    "summary": self.code_analyzer.generate_code_summary(
                        doc.page_content,
                        language
                    )
                })
        
        # 3. 生成回答
        prompt = f"""基于以下代码信息回答问题。

问题：{question}

代码上下文：
"""
        for ctx in code_contexts:
            prompt += f"\n文件：{ctx['file']}\n"
            prompt += f"摘要：{ctx['summary']}\n"
            prompt += f"代码：\n```\n{ctx['content'][:500]}...\n```\n"
        
        response = self.llm.invoke(prompt)
        
        return {
            "answer": response.content,
            "code_files": [ctx["file"] for ctx in code_contexts],
            "structures": [ctx["structure"] for ctx in code_contexts]
        }
    
    def _detect_language(self, file_path: str) -> str:
        """检测编程语言"""
        ext_map = {
            ".py": "python",
            ".java": "java",
            ".js": "javascript",
            ".ts": "javascript",
            ".go": "go"
        }
        
        ext = os.path.splitext(file_path)[1]
        return ext_map.get(ext, "unknown")
```

#### 难点 3：答案准确性验证

**问题**：
- LLM 可能产生幻觉，给出不存在的信息
- 如何验证答案确实来自文档
- 如何处理文档信息不足的情况

**解决方案**：
```python
class AnswerValidator:
    """答案验证器"""
    
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
    
    def validate_answer(
        self,
        question: str,
        answer: str,
        source_docs: List[str]
    ) -> Dict[str, Any]:
        """验证答案"""
        
        # 1. 检查答案是否基于文档
        grounding_check = self._check_grounding(answer, source_docs)
        
        # 2. 检查答案完整性
        completeness_check = self._check_completeness(question, answer)
        
        # 3. 检查答案一致性
        consistency_check = self._check_consistency(answer, source_docs)
        
        # 综合评分
        confidence_score = (
            grounding_check["score"] * 0.5 +
            completeness_check["score"] * 0.3 +
            consistency_check["score"] * 0.2
        )
        
        return {
            "is_valid": confidence_score > 0.7,
            "confidence": confidence_score,
            "checks": {
                "grounding": grounding_check,
                "completeness": completeness_check,
                "consistency": consistency_check
            },
            "suggestions": self._generate_suggestions(
                confidence_score,
                grounding_check,
                completeness_check
            )
        }
    
    def _check_grounding(self, answer: str, source_docs: List[str]) -> Dict[str, Any]:
        """检查答案是否基于文档"""
        
        prompt = f"""检查答案中的每个陈述是否都能在源文档中找到依据。

答案：
{answer}

源文档：
{chr(10).join(source_docs)}

请评估：
1. 答案中有多少陈述有文档支持
2. 是否有无依据的陈述
3. 给出0-1的评分

以JSON格式返回：
{{
    "supported_statements": 数量,
    "unsupported_statements": 数量,
    "score": 0-1,
    "details": "说明"
}}
"""
        
        response = self.llm.invoke(prompt)
        try:
            result = json.loads(response.content)
            return result
        except:
            return {"score": 0.5, "details": "验证失败"}
    
    def _check_completeness(self, question: str, answer: str) -> Dict[str, Any]:
        """检查答案完整性"""
        
        prompt = f"""评估答案是否完整回答了问题。

问题：{question}
答案：{answer}

评估标准：
1. 是否直接回答了问题
2. 是否遗漏了重要信息
3. 是否提供了足够的细节

以JSON格式返回：
{{
    "score": 0-1,
    "missing_aspects": ["缺失的方面"],
    "details": "说明"
}}
"""
        
        response = self.llm.invoke(prompt)
        try:
            return json.loads(response.content)
        except:
            return {"score": 0.5, "missing_aspects": []}
    
    def _check_consistency(self, answer: str, source_docs: List[str]) -> Dict[str, Any]:
        """检查答案一致性"""
        
        # 检查答案内部是否有矛盾
        # 检查答案与文档是否有冲突
        
        prompt = f"""检查答案是否存在内部矛盾或与文档冲突。

答案：{answer}

源文档：
{chr(10).join(source_docs[:3])}

以JSON格式返回：
{{
    "score": 0-1,
    "contradictions": ["发现的矛盾"],
    "details": "说明"
}}
"""
        
        response = self.llm.invoke(prompt)
        try:
            return json.loads(response.content)
        except:
            return {"score": 0.8, "contradictions": []}
    
    def _generate_suggestions(
        self,
        confidence: float,
        grounding: Dict,
        completeness: Dict
    ) -> List[str]:
        """生成改进建议"""
        
        suggestions = []
        
        if confidence < 0.7:
            suggestions.append("答案置信度较低，建议人工审核")
        
        if grounding.get("unsupported_statements", 0) > 0:
            suggestions.append("答案包含无文档支持的陈述，请核实")
        
        if completeness.get("missing_aspects"):
            suggestions.append(
                f"答案可能遗漏了以下方面：{', '.join(completeness['missing_aspects'])}"
            )
        
        return suggestions


class ValidatedQAAgent:
    """带验证的问答 Agent"""
    
    def __init__(self, vectorstore: Chroma):
        self.qa_agent = EnterpriseQAAgent(vectorstore)
        self.validator = AnswerValidator()
    
    def query_with_validation(self, question: str) -> Dict[str, Any]:
        """带验证的查询"""
        
        # 1. 获取答案
        result = self.qa_agent.query(question)
        
        # 2. 验证答案
        source_texts = [doc["content"] for doc in result["sources"]]
        validation = self.validator.validate_answer(
            question,
            result["answer"],
            source_texts
        )
        
        # 3. 根据验证结果决定响应
        if validation["is_valid"]:
            return {
                "answer": result["answer"],
                "sources": result["sources"],
                "confidence": validation["confidence"],
                "validated": True
            }
        else:
            # 置信度低，提供警告
            return {
                "answer": result["answer"],
                "sources": result["sources"],
                "confidence": validation["confidence"],
                "validated": False,
                "warning": "答案可能不够准确，建议：",
                "suggestions": validation["suggestions"]
            }
```

### 2.5 生产环境实践

#### 性能优化

```python
class OptimizedKnowledgeBase:
    """优化的知识库"""
    
    def __init__(self):
        # 使用更高效的向量数据库
        self.vectorstore = self._init_pinecone()
        
        # 多级缓存
        self.l1_cache = {}  # 内存缓存
        self.l2_cache = redis.Redis()  # Redis缓存
        
        # 查询优化
        self.query_optimizer = QueryOptimizer()
    
    def _init_pinecone(self):
        """初始化 Pinecone（生产级向量数据库）"""
        import pinecone
        
        pinecone.init(
            api_key=os.getenv("PINECONE_API_KEY"),
            environment="us-west1-gcp"
        )
        
        index_name = "enterprise-kb"
        
        if index_name not in pinecone.list_indexes():
            pinecone.create_index(
                name=index_name,
                dimension=1536,  # OpenAI embeddings
                metric="cosine",
                pods=2,  # 2个pod提高性能
                replicas=2  # 2个副本提高可用性
            )
        
        return pinecone.Index(index_name)
    
    def query_optimized(self, question: str) -> Dict[str, Any]:
        """优化的查询"""
        
        # 1. 检查L1缓存
        cache_key = hashlib.md5(question.encode()).hexdigest()
        if cache_key in self.l1_cache:
            logger.info("L1缓存命中")
            return self.l1_cache[cache_key]
        
        # 2. 检查L2缓存
        cached = self.l2_cache.get(f"kb:{cache_key}")
        if cached:
            logger.info("L2缓存命中")
            result = json.loads(cached)
            self.l1_cache[cache_key] = result
            return result
        
        # 3. 查询优化
        optimized_query = self.query_optimizer.optimize(question)
        
        # 4. 执行查询
        result = self._execute_query(optimized_query)
        
        # 5. 保存缓存
        self.l2_cache.setex(
            f"kb:{cache_key}",
            3600,  # 1小时
            json.dumps(result)
        )
        self.l1_cache[cache_key] = result
        
        return result


class QueryOptimizer:
    """查询优化器"""
    
    def optimize(self, query: str) -> str:
        """优化查询"""
        
        # 1. 查询扩展（添加同义词）
        expanded = self._expand_query(query)
        
        # 2. 查询重写（改写为更精确的形式）
        rewritten = self._rewrite_query(expanded)
        
        # 3. 添加上下文
        contextualized = self._add_context(rewritten)
        
        return contextualized
    
    def _expand_query(self, query: str) -> str:
        """查询扩展"""
        # 添加同义词、相关词
        synonyms = {
            "部署": ["发布", "上线", "deploy"],
            "配置": ["设置", "config", "configuration"]
        }
        
        expanded_terms = []
        for word, syns in synonyms.items():
            if word in query:
                expanded_terms.extend(syns)
        
        if expanded_terms:
            return f"{query} {' '.join(expanded_terms)}"
        
        return query
```

### 2.6 实际效果

```python
# 真实数据（基于某科技公司实施案例）
KNOWLEDGE_BASE_METRICS = {
    "before": {
        "avg_search_time": 300,  # 5分钟（人工搜索）
        "success_rate": 0.60,    # 60%能找到答案
        "employee_satisfaction": 3.2,
        "support_tickets": 500   # 月均IT支持工单
    },
    "after": {
        "avg_search_time": 10,   # 10秒
        "success_rate": 0.85,    # 85%能找到答案
        "employee_satisfaction": 4.5,
        "support_tickets": 150,  # 减少70%
        "cost_savings": "$50K/month"  # 节省人力成本
    },
    "kb_stats": {
        "total_documents": 15000,
        "total_chunks": 150000,
        "daily_queries": 2000,
        "avg_response_time": "2.5s",
        "cache_hit_rate": 0.65
    }
}
```



---

## 案例 3：自动化运维 Agent

### 3.1 业务场景

某互联网公司有数百台服务器需要运维管理,人工操作效率低、易出错。需要构建自动化运维 Agent:
- 自动监控服务器状态
- 智能故障诊断与修复
- 自动化部署与回滚
- 日志分析与告警

### 3.2 系统架构

```
监控数据源
  ├─ Prometheus 指标
  ├─ ELK 日志
  ├─ APM 追踪
  └─ 告警系统

        ↓

运维 Agent
  ├─ 监控分析
  ├─ 故障诊断
  ├─ 自动修复
  └─ 决策引擎

        ↓

执行层
  ├─ Ansible
  ├─ Kubernetes
  ├─ Shell 脚本
  └─ API 调用
```

### 3.3 核心实现

```python
from typing import Dict, Any, List, Optional
from langchain_openai import ChatOpenAI
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain.tools import Tool
from langchain.prompts import ChatPromptTemplate, MessagesPlaceholder
import subprocess
import requests
import logging
from datetime import datetime, timedelta
import json

logger = logging.getLogger(__name__)


class MonitoringService:
    """监控服务"""
    
    def __init__(self):
        self.prometheus_url = "http://prometheus:9090"
        self.elasticsearch_url = "http://elasticsearch:9200"
    
    def query_metrics(self, query: str, time_range: str = "5m") -> Dict[str, Any]:
        """查询 Prometheus 指标"""
        try:
            response = requests.get(
                f"{self.prometheus_url}/api/v1/query",
                params={
                    "query": query,
                    "time": datetime.now().isoformat()
                },
                timeout=10
            )
            
            if response.status_code == 200:
                data = response.json()
                return {
                    "success": True,
                    "data": data.get("data", {}).get("result", [])
                }
            else:
                return {"success": False, "error": f"HTTP {response.status_code}"}
                
        except Exception as e:
            logger.error(f"查询指标失败: {str(e)}")
            return {"success": False, "error": str(e)}
    
    def search_logs(self, query: str, time_range: str = "5m") -> List[Dict]:
        """搜索 Elasticsearch 日志"""
        try:
            now = datetime.now()
            time_from = now - timedelta(minutes=int(time_range.rstrip('m')))
            
            search_body = {
                "query": {
                    "bool": {
                        "must": [
                            {"query_string": {"query": query}},
                            {
                                "range": {
                                    "@timestamp": {
                                        "gte": time_from.isoformat(),
                                        "lte": now.isoformat()
                                    }
                                }
                            }
                        ]
                    }
                },
                "size": 100,
                "sort": [{"@timestamp": "desc"}]
            }
            
            response = requests.post(
                f"{self.elasticsearch_url}/logs-*/_search",
                json=search_body,
                timeout=10
            )
            
            if response.status_code == 200:
                hits = response.json().get("hits", {}).get("hits", [])
                return [hit["_source"] for hit in hits]
            else:
                return []
                
        except Exception as e:
            logger.error(f"搜索日志失败: {str(e)}")
            return []
    
    def get_server_status(self, server_ip: str) -> Dict[str, Any]:
        """获取服务器状态"""
        # 查询 CPU、内存、磁盘使用率
        metrics = {
            "cpu": self.query_metrics(f'node_cpu_seconds_total{{instance="{server_ip}"}}'),
            "memory": self.query_metrics(f'node_memory_MemAvailable_bytes{{instance="{server_ip}"}}'),
            "disk": self.query_metrics(f'node_filesystem_avail_bytes{{instance="{server_ip}"}}')
        }
        
        return {
            "server": server_ip,
            "timestamp": datetime.now().isoformat(),
            "metrics": metrics
        }


class AutomationExecutor:
    """自动化执行器"""
    
    def __init__(self):
        self.ansible_inventory = "/etc/ansible/hosts"
    
    def execute_ansible_playbook(
        self,
        playbook_path: str,
        hosts: str,
        extra_vars: Optional[Dict] = None
    ) -> Dict[str, Any]:
        """执行 Ansible Playbook"""
        
        cmd = [
            "ansible-playbook",
            playbook_path,
            "-i", self.ansible_inventory,
            "-l", hosts
        ]
        
        if extra_vars:
            cmd.extend(["--extra-vars", json.dumps(extra_vars)])
        
        try:
            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                timeout=300
            )
            
            return {
                "success": result.returncode == 0,
                "stdout": result.stdout,
                "stderr": result.stderr,
                "return_code": result.returncode
            }
            
        except subprocess.TimeoutExpired:
            return {"success": False, "error": "执行超时"}
        except Exception as e:
            logger.error(f"执行 Playbook 失败: {str(e)}")
            return {"success": False, "error": str(e)}
    
    def restart_service(self, server: str, service_name: str) -> Dict[str, Any]:
        """重启服务"""
        playbook = f"""
---
- hosts: {server}
  tasks:
    - name: Restart service
      systemd:
        name: {service_name}
        state: restarted
"""
        
        # 写入临时 playbook
        playbook_path = f"/tmp/restart_{service_name}.yml"
        with open(playbook_path, 'w') as f:
            f.write(playbook)
        
        return self.execute_ansible_playbook(playbook_path, server)



class OpsAgent:
    """运维 Agent"""
    
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
        self.monitoring = MonitoringService()
        self.executor = AutomationExecutor()
        self.tools = self._create_tools()
        self.agent = self._create_agent()
    
    def _create_tools(self) -> List[Tool]:
        """创建工具"""
        return [
            Tool(
                name="check_server_status",
                func=lambda server: json.dumps(self.monitoring.get_server_status(server)),
                description="检查服务器状态,输入服务器IP"
            ),
            Tool(
                name="search_error_logs",
                func=lambda query: json.dumps(self.monitoring.search_logs(f"level:ERROR AND {query}")),
                description="搜索错误日志,输入搜索关键词"
            ),
            Tool(
                name="restart_service",
                func=lambda args: json.dumps(self.executor.restart_service(*args.split(','))),
                description="重启服务,输入格式: server_ip,service_name"
            )
        ]
    
    def diagnose_and_fix(self, alert: Dict[str, Any]) -> Dict[str, Any]:
        """诊断并修复问题"""
        
        prompt = f"""你是一个运维专家。收到以下告警:

告警信息: {json.dumps(alert, ensure_ascii=False)}

请按以下步骤处理:
1. 分析告警原因
2. 使用工具检查相关服务器和日志
3. 判断是否需要自动修复
4. 如果需要,执行修复操作
5. 总结处理结果

注意: 只有在确认安全的情况下才执行自动修复。
"""
        
        result = self.agent.invoke({"input": prompt})
        return result
```

### 3.4 核心难点

#### 难点 1: 安全性控制

**问题**: 自动化操作可能造成严重后果

**解决方案**:
- 操作前二次确认机制
- 高危操作需要人工审批
- 完整的操作日志和回滚能力
- 沙箱环境测试

#### 难点 2: 故障诊断准确性

**问题**: 复杂系统故障原因多样

**解决方案**:
- 建立故障知识库
- 多维度数据关联分析
- 历史案例学习
- 专家规则 + AI 推理结合

---

## 案例 4-7 概要

由于篇幅限制,以下案例提供核心要点:

### 案例 4: 智能数据分析平台

**场景**: 自动化数据分析和报告生成
**核心技术**: Pandas Agent + 数据可视化 + 自然语言查询
**难点**: SQL生成准确性、大数据量处理、图表自动选择

### 案例 5: 代码审查与重构助手

**场景**: 自动代码审查、重构建议、代码质量提升
**核心技术**: AST分析 + 静态分析工具 + LLM代码理解
**难点**: 代码上下文理解、重构安全性、编程规范适配

### 案例 6: 多语言翻译与本地化系统

**场景**: 软件国际化、文档翻译、术语一致性
**核心技术**: 翻译记忆库 + 术语管理 + 上下文感知翻译
**难点**: 专业术语准确性、格式保持、批量处理效率

### 案例 7: 智能招聘筛选系统

**场景**: 简历筛选、候选人匹配、面试问题生成
**核心技术**: 简历解析 + 技能匹配 + 多维度评估
**难点**: 公平性保证、隐私保护、评估标准量化

---

## 总结与最佳实践

### 通用经验

1. **从简单场景开始**: 先解决明确、可量化的问题
2. **人机协作**: AI辅助而非完全替代人工
3. **持续优化**: 收集反馈,不断改进
4. **安全第一**: 高危操作必须有人工审核
5. **可观测性**: 完善的日志、监控、告警

### 技术选型建议

| 场景 | 推荐框架 | 向量数据库 | LLM模型 |
|------|---------|-----------|---------|
| 客服系统 | LangChain | Chroma | GPT-4 |
| 知识库 | LlamaIndex | Pinecone | GPT-4 |
| 运维自动化 | 自定义 | - | GPT-4 |
| 数据分析 | PandasAI | - | GPT-4 |

### 成本优化

1. 使用缓存减少API调用
2. 简单问题用GPT-3.5
3. 批量处理降低成本
4. 本地模型处理敏感数据

---

> **@author erik.zhou**
> 
> 本文档持续更新,欢迎反馈改进建议
