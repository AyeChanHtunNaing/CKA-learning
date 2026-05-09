# The Modern Microservice — Notes

## 1. Main Idea

**Modern Microservice** ဆိုတာ application ကြီးတစ်ခုကို feature/function အသေးလေးတွေ ခွဲပြီး service သီးသန့်တွေအဖြစ် တည်ဆောက်ထားတဲ့ architecture ဖြစ်ပါတယ်။

Monolith မှာ feature အားလုံး application တစ်ခုတည်းထဲမှာ စုပြီးရှိပေမယ့်၊ Microservices မှာတော့ feature တစ်ခုချင်းစီကို service သီးသန့်အနေနဲ့ ခွဲထားပါတယ်။

ဥပမာ — E-commerce system တစ်ခုမှာ

- User Service
- Product Service
- Order Service
- Payment Service
- Notification Service
- Delivery Service

ဆိုပြီး service တွေခွဲထားနိုင်ပါတယ်။

---

## 2. Pebbles vs Boulder Analogy

စာထဲမှာ **pebbles** နဲ့ **1000-ton boulder** ဥပမာသုံးထားပါတယ်။

### Boulder

**Boulder** ဆိုတာ ကျောက်တုံးကြီးတစ်လုံးပါ။ ၎င်းက **Monolithic Application** ကို ကိုယ်စားပြုပါတယ်။

Monolith app တစ်ခုမှာ feature တွေအကုန်လုံး တစ်နေရာတည်းမှာ စုပြီးရှိတာကြောင့် ရွှေ့ပြောင်းဖို့၊ ပြင်ဆင်ဖို့၊ scale လုပ်ဖို့ ခက်ပါတယ်။

### Pebbles

**Pebbles** ဆိုတာ ကျောက်စရစ်လေးတွေပါ။ ၎င်းတွေက **Microservices** ကို ကိုယ်စားပြုပါတယ်။

Monolith ကြီးထဲက business function တွေကို သေးသေးလေးတွေ ခွဲထုတ်လိုက်တဲ့အခါ microservices ဖြစ်လာပါတယ်။ Pebble တစ်လုံးချင်းစီကို ကိုင်တွယ်ရတာ လွယ်သလို microservice တစ်ခုချင်းစီကို deploy, update, scale လုပ်ရတာလည်း ပိုလွယ်ပါတယ်။

---

## 3. Microservice ဆိုတာဘာလဲ?

Microservice ဆိုတာ **specific business function တစ်ခုကိုပဲ တာဝန်ယူတဲ့ independent service အသေးလေး** ဖြစ်ပါတယ်။

ဥပမာ — Payment Service ဆိုရင် payment နဲ့ပတ်သက်တဲ့ logic ကိုပဲ တာဝန်ယူမယ်။ Notification Service ဆိုရင် email, SMS, push notification ပို့တာကိုပဲ တာဝန်ယူမယ်။

အဓိကအချက်က service တစ်ခုချင်းစီက independent ဖြစ်ပြီး loosely coupled ဖြစ်ပါတယ်။

---

## 4. Loosely Coupled ဖြစ်ခြင်း

Microservices တွေက **loosely coupled** ဖြစ်ပါတယ်။ ဆိုလိုတာက service တစ်ခုက အခြား service တစ်ခုအပေါ် အလွန်အကျွံ မမှီခိုဘူးဆိုတဲ့အဓိပ္ပါယ်ပါ။

ဥပမာ — Order Service က Payment Service ကို API မှတစ်ဆင့်ခေါ်နိုင်ပေမယ့် Payment Service ရဲ့ internal code ကို တိုက်ရိုက်မမှီခိုပါ။

ဒါကြောင့် Payment Service ကို update လုပ်ချင်ရင် Order Service တစ်ခုလုံးကို ပြန်ရေးစရာမလိုပါ။

---

## 5. Individual Deployment

Microservices တွေကို service တစ်ခုချင်းစီ သီးသန့် deploy လုပ်နိုင်ပါတယ်။

ဥပမာ — Payment Service ကို update လုပ်ချင်ရင် Payment Service တစ်ခုတည်းကိုပဲ deploy လုပ်နိုင်ပါတယ်။ Application တစ်ခုလုံးကို restart လုပ်စရာမလိုပါ။

```text
Payment Service  → deploy separately
Order Service    → unchanged
User Service     → unchanged
Notification     → unchanged
```

ဒီလို individual deployment လုပ်နိုင်တာကြောင့် update လုပ်ရတာ ပိုမြန်ပြီး risk လည်းနည်းပါတယ်။

---

## 6. Resource Usage ပိုသက်သာခြင်း

Monolith application မှာ application တစ်ခုလုံးကို server ကြီးတစ်လုံးပေါ်မှာ run ရတာများပါတယ်။ ဒါကြောင့် CPU, RAM, storage, network capacity အများကြီးလိုနိုင်ပါတယ်။

Microservices မှာတော့ service တစ်ခုချင်းစီအတွက် လိုအပ်သလောက် resource ပဲ ပေးနိုင်ပါတယ်။

ဥပမာ —

```text
Payment Service       → high CPU / security focused
Notification Service  → moderate CPU / queue based
Report Service        → high memory
User Service          → normal resources
```

ဒီလိုလိုအပ်သလောက် resource ပေးနိုင်တဲ့အတွက် compute cost လျော့နိုင်ပါတယ်။

---

## 7. API Communication

Microservices တွေက တစ်ခုနဲ့တစ်ခု network ပေါ်ကနေ **API** တွေသုံးပြီး ဆက်သွယ်ကြပါတယ်။

ဥပမာ —

```text
Frontend → Order Service → Payment Service
                         → Inventory Service
                         → Notification Service
```

API တွေက internal services တွေကြား communication လုပ်ဖို့သုံးနိုင်သလို external third-party services တွေနဲ့လည်း ချိတ်ဆက်ဖို့သုံးနိုင်ပါတယ်။

ဥပမာ — Payment Service က Stripe, PayPal, KBZPay စတဲ့ external payment provider API တွေနဲ့ ချိတ်ဆက်နိုင်ပါတယ်။

---

## 8. Event-driven Architecture နဲ့ SOA

Microservices architecture က **Event-driven Architecture** နဲ့ **Service-Oriented Architecture (SOA)** principles တွေနဲ့လည်း ကိုက်ညီပါတယ်။

### Event-driven Architecture

Event တစ်ခုဖြစ်လာတဲ့အခါ အခြား service တွေက အဲဒီ event ကိုနားထောင်ပြီး လုပ်ဆောင်ကြတာပါ။

ဥပမာ —

```text
Order Created Event
        ↓
Payment Service processes payment
        ↓
Inventory Service reduces stock
        ↓
Notification Service sends confirmation email
```

### SOA

SOA ဆိုတာ complex application ကို independent services တွေနဲ့ ဖွဲ့စည်းတဲ့ principle ဖြစ်ပါတယ်။ Microservices က SOA idea ကို ပိုသေးငယ်ပြီး modern ပုံစံနဲ့ အသုံးချထားတာလို့နားလည်နိုင်ပါတယ်။

---

## 9. Different Programming Languages သုံးနိုင်ခြင်း

Microservice တစ်ခုချင်းစီကို သင့်တော်တဲ့ programming language နဲ့ရေးနိုင်ပါတယ်။

ဥပမာ —

```text
User Service        → Java / Spring Boot
Payment Service     → Go / Java
Recommendation      → Python
Realtime Chat       → Node.js
Report Service      → Python / Java
```

Monolith မှာတော့ app တစ်ခုလုံး language တစ်မျိုးတည်းနဲ့ရေးထားတာများပါတယ်။ Microservices မှာတော့ service ရဲ့ business function နဲ့လိုက်ဖက်တဲ့ language ကိုရွေးနိုင်တာကြောင့် flexibility ပိုများပါတယ်။

---

## 10. Commodity Hardware ပေါ်မှာ Deploy လုပ်နိုင်ခြင်း

Microservices တွေက service တစ်ခုချင်းစီ resource requirement နည်းနိုင်တာကြောင့် expensive server ကြီးတွေမလိုဘဲ inexpensive commodity hardware ပေါ်မှာ deploy လုပ်နိုင်ပါတယ်။

ဆိုလိုတာက server ကြီးတစ်လုံးဝယ်ပြီး application တစ်ခုလုံး run မလုပ်ဘဲ service သေးသေးလေးတွေကို server သေးသေးလေးများပေါ်မှာ ဖြန့်ပြီး run လုပ်နိုင်ပါတယ်။

---

## 11. Scalability အားသာချက်

Microservices ရဲ့ အကြီးမားဆုံး advantage တစ်ခုက **scalability** ဖြစ်ပါတယ်။

Monolith မှာ traffic များလာရင် application တစ်ခုလုံးကို scale လုပ်ရပါတယ်။ Microservices မှာတော့ traffic များတဲ့ service တစ်ခုတည်းကိုပဲ scale လုပ်နိုင်ပါတယ်။

ဥပမာ — Payment traffic များလာရင် Payment Service ကိုပဲ instance ထပ်တိုးနိုင်ပါတယ်။

```text
Payment Service x 5
Order Service x 2
User Service x 2
Notification Service x 1
```

ဒီလို individual scaling လုပ်နိုင်တာကြောင့် resource waste နည်းပြီး cost-effective ဖြစ်ပါတယ်။

---

## 12. Autoscaling

Microservices တွေကို demand ပေါ်မူတည်ပြီး automatically scale လုပ်နိုင်ပါတယ်။

ဥပမာ — CPU usage 80% ကျော်သွားရင် Payment Service instance အသစ်တွေ auto-create လုပ်မယ်။ Traffic ကျသွားရင် instance တွေပြန်လျှော့မယ်။

ဒါကို demand-based autoscaling လို့ခေါ်ပါတယ်။ Cloud-native system တွေမှာ Kubernetes, AWS Auto Scaling, ECS, Cloud Run စတာတွေနဲ့ ပြုလုပ်နိုင်ပါတယ်။

---

## 13. Seamless Upgrade and Patching

Microservices architecture မှာ service တစ်ခုချင်းစီကို တဖြည်းဖြည်း update လုပ်နိုင်ပါတယ်။

Monolith မှာ application တစ်ခုလုံးကို rebuild, recompile, restart လုပ်ရနိုင်ပေမယ့် microservices မှာ service တစ်ခုချင်းစီကိုသာ update လုပ်လို့ရပါတယ်။

ဥပမာ —

```text
Step 1: Update Payment Service v1 → v2
Step 2: Keep Order Service running
Step 3: Keep User Service running
Step 4: Roll out gradually
```

ဒီလိုလုပ်နိုင်တဲ့အတွက် downtime နည်းပြီး customer တွေအတွက် service disruption လည်းနည်းပါတယ်။

---

## 14. Faster Feature Development

Microservices မှာ team တွေကို service အလိုက်ခွဲပြီးအလုပ်လုပ်နိုင်ပါတယ်။

ဥပမာ —

```text
Team A → User Service
Team B → Payment Service
Team C → Order Service
Team D → Notification Service
```

Team တစ်ခုက feature အသစ်တစ်ခုလုပ်နေရင် အခြား team တွေကို အရမ်းမစောင့်ရပါ။ ဒါကြောင့် agile development နဲ့ပိုကိုက်ညီပြီး feature အသစ်တွေကိုပိုမြန်မြန် release လုပ်နိုင်ပါတယ်။

---

## 15. Microservices ရဲ့ Complexity

Microservices က အားသာချက်များပေမယ့် complexity လည်းရှိပါတယ်။

Service တွေများလာတာကြောင့် —

- Network communication ကိုစီမံရမယ်
- API versioning လုပ်ရမယ်
- Service discovery လိုနိုင်တယ်
- Logging နဲ့ monitoring ပိုရှုပ်နိုင်တယ်
- Distributed tracing လိုလာနိုင်တယ်
- Data consistency ကိုဂရုစိုက်ရမယ်
- Deployment pipeline ကောင်းကောင်းလိုတယ်

ဒါကြောင့် microservices က simple system အတွက်မဟုတ်ဘဲ large-scale, fast-changing applications တွေအတွက် ပိုသင့်တော်ပါတယ်။

---

## 16. Monolith vs Microservices

| Area | Monolith | Microservices |
|---|---|---|
| Structure | One large application | Many small services |
| Deployment | Deploy entire app | Deploy service individually |
| Scaling | Scale whole app | Scale only required service |
| Technology | Usually one language/framework | Different languages possible |
| Downtime | Higher chance during updates | Lower with rolling updates |
| Team structure | Teams may conflict on same codebase | Teams can own separate services |
| Hardware | Often requires large server | Can use smaller commodity servers |
| Complexity | Simpler to start | More complex distributed system |

---

## 17. Simple Architecture Diagram

```text
                 Users / Clients
                       ↓
                 API Gateway
                       ↓
   ┌───────────────┬───────────────┬───────────────┐
   ↓               ↓               ↓               ↓
User Service   Order Service   Payment Service   Product Service
   ↓               ↓               ↓               ↓
User DB        Order DB        Payment DB        Product DB
                       ↓
              Notification Service
```

ဒီ diagram မှာ service တစ်ခုချင်းစီက သီးသန့်တာဝန်ရှိပြီး API Gateway က client request တွေကို သင့်တော်တဲ့ service ဆီပို့ပေးပါတယ်။

---

## 18. Key Takeaways

- Microservices ဆိုတာ monolith application ကြီးကို service အသေးလေးတွေ ခွဲထားတဲ့ architecture ဖြစ်ပါတယ်။
- Service တစ်ခုချင်းစီက specific business function တစ်ခုကို တာဝန်ယူပါတယ်။
- Service တွေက API တွေသုံးပြီး network ပေါ်မှာ ဆက်သွယ်ကြပါတယ်။
- Microservices တွေကို individually deploy, update, scale လုပ်နိုင်ပါတယ်။
- Resource usage ပိုထိရောက်ပြီး compute cost လျော့နိုင်ပါတယ်။
- Different programming languages နဲ့ service တစ်ခုချင်းစီကိုရေးနိုင်ပါတယ်။
- Autoscaling နဲ့ rolling update တွေကြောင့် downtime နည်းနိုင်ပါတယ်။
- Team တွေကို service အလိုက်ခွဲပြီး agile ပုံစံနဲ့မြန်မြန် develop လုပ်နိုင်ပါတယ်။
- ဒါပေမယ့် distributed system ဖြစ်တဲ့အတွက် monitoring, logging, API management, network failure, data consistency စတဲ့ complexity တွေလည်းတိုးလာပါတယ်။

---

## 19. One-line Summary

**Modern Microservice architecture ဆိုတာ application ကြီးတစ်ခုကို independent service အသေးလေးတွေခွဲပြီး service တစ်ခုချင်းစီကို သီးသန့် develop, deploy, scale, update လုပ်နိုင်အောင် တည်ဆောက်တဲ့ modern software architecture ဖြစ်ပါတယ်။**
