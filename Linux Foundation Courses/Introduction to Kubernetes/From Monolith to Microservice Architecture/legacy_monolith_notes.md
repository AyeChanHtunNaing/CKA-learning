# The Legacy Monolith - Notes

## 1. Legacy Monolith ဆိုတာဘာလဲ?

**Legacy Monolith Application** ဆိုတာ အရင်ခေတ်ကတည်းကရေးထားတဲ့ application ကြီးတစ်ခုဖြစ်ပြီး feature တွေ၊ business logic တွေ၊ database logic တွေကို **application တစ်ခုတည်းထဲမှာ စုပေါင်းရေးထားတဲ့ software architecture** ဖြစ်ပါတယ်။

ဥပမာ - Hotel Booking System တစ်ခုမှာ:

- Login
- Room Booking
- Payment
- Customer Management
- Report
- Admin Panel

ဒီ feature တွေအကုန်လုံးကို service သီးသန့်မခွဲဘဲ **application တစ်ခုတည်းထဲမှာ စုပြီးရေးထားတာ** ကို Monolith လို့ခေါ်နိုင်ပါတယ်။

---

## 2. Pebbles and Boulder ဥပမာ

စာထဲမှာ legacy monolith ကို **1000-ton boulder** နဲ့ နှိုင်းယှဉ်ထားပါတယ်။

### Pebbles

Pebbles ဆိုတာ ကျောက်သေးသေးလေးတွေပါ။ Bucket ထဲထည့်ပြီး ဘယ်နေရာမဆို သယ်သွားရလွယ်ပါတယ်။

ဒါက **cloud-ready application** တွေကို ဆိုလိုပါတယ်။

ဥပမာ:

- Microservices
- Containerized applications
- Cloud-native applications

### Boulder

Boulder ဆိုတာ ကျောက်တုံးကြီးပါ။ အလေးချိန်ကြီးလို့ သယ်ရခက်ပါတယ်။

ဒါက **Legacy Monolith Application** ကို ဆိုလိုပါတယ်။

ဘာကြောင့်လဲဆိုတော့:

- Codebase အရမ်းကြီးတယ်
- Architecture ဟောင်းတယ်
- Programming language / framework ဟောင်းနေနိုင်တယ်
- Redundant logic တွေများတယ်
- Cloud ပေါ်ရွှေ့ဖို့ မလွယ်ဘူး

---

## 3. Monolith Application ရဲ့ ပြဿနာများ

### 3.1 Code Complexity တိုးလာခြင်း

အချိန်ကြာလာတာနဲ့ application ထဲကို feature အသစ်တွေ ထပ်ထည့်လာပါတယ်။

ဥပမာ:

```text
Version 1: Login + User Management
Version 2: Payment
Version 3: Reports
Version 4: Notification
Version 5: Admin Dashboard
```

ဒီလိုနဲ့ application တစ်ခုလုံး ကြီးလာပြီး code complexity တိုးလာပါတယ်။

ဖြစ်လာနိုင်တဲ့ ပြဿနာတွေက:

- Code ဖတ်ရခက်လာတယ်
- Bug ပြင်ရခက်လာတယ်
- Feature အသစ်ထည့်ရခက်လာတယ်
- Loading time တိုးလာတယ်
- Compiling time တိုးလာတယ်
- Build time ကြာလာတယ်
- Testing လုပ်ရတာ ကြာလာတယ်

---

### 3.2 Administration အနည်းငယ်လွယ်ပေမယ့် Risk ရှိခြင်း

Monolith application က server တစ်လုံးပေါ်မှာ application တစ်ခုတည်းအနေနဲ့ run နိုင်ပါတယ်။

```text
One Application
      ↓
One Server / VM / Mainframe
```

ဒါကြောင့် administration ပိုင်းမှာ တစ်နေရာတည်းကိုပဲ maintain လုပ်ရလို့ လွယ်သလိုထင်ရပါတယ်။

ဒါပေမယ့် application ကြီးလာတာနဲ့ server requirement တွေလည်း ကြီးလာပါတယ်။

---

### 3.3 Hardware အရမ်းစားခြင်း

Monolith application တစ်ခုလုံးက process တစ်ခုအနေနဲ့ run တာဖြစ်တဲ့အတွက် server တစ်လုံးတည်းမှာ resource အများကြီးလိုပါတယ်။

လိုအပ်နိုင်တဲ့ resources တွေက:

- CPU
- Memory / RAM
- Storage
- Network capacity

ဒီလို hardware ကြီးတွေက:

- ဈေးကြီးတယ်
- ဝယ်ယူဖို့ခက်တယ်
- Procure လုပ်ဖို့အချိန်ကြာတယ်
- Maintenance cost များတယ်

---

### 3.4 Feature တစ်ခုချင်းစီကို Scale လုပ်မရခြင်း

Monolith application မှာ feature တွေအကုန်လုံး application တစ်ခုတည်းထဲမှာရှိပါတယ်။

ဥပမာ payment feature တစ်ခုတည်း traffic များလာတယ်ဆိုပါစို့။

Microservices ဖြစ်ရင် payment service ကိုပဲ scale လုပ်လို့ရပါတယ်။

```text
Payment Service → Scale independently
```

ဒါပေမယ့် Monolith မှာ payment feature ကိုပဲ ခွဲပြီး scale လုပ်လို့မရပါဘူး။ Application တစ်ခုလုံးကိုပဲ ထပ် run ရပါတယ်။

```text
Server 1: Full Monolith App
Server 2: Full Monolith App
Server 3: Full Monolith App
```

Payment feature ပဲ busy ဖြစ်နေပေမယ့် app တစ်ခုလုံးကို duplicate လုပ်ရတဲ့အတွက် resource waste ဖြစ်နိုင်ပါတယ်။

---

### 3.5 Load Balancer နောက်မှာ Instance အသစ်များ ထပ်တင်ရခြင်း

Monolith application ကို scale လုပ်ချင်ရင် server အသစ်တွေထပ်တင်ပြီး application တစ်ခုလုံးကို ထပ် deploy လုပ်ရပါတယ်။

```text
Users
  ↓
Load Balancer
  ↓
Server 1 - Monolith App
Server 2 - Monolith App
Server 3 - Monolith App
```

ဒီ approach မှာ:

- Load balancer လိုတယ်
- Server တွေ ထပ်လိုတယ်
- Application တစ်ခုလုံးကို duplicate လုပ်ရတယ်
- Cost တိုးတယ်
- Maintenance ပိုခက်တယ်

---

### 3.6 Upgrade / Patch / Migration လုပ်ရင် Downtime ဖြစ်နိုင်ခြင်း

Monolith application ကို update လုပ်ချင်ရင် application တစ်ခုလုံးကို restart လုပ်ရတတ်ပါတယ်။

ဥပမာ:

- Version upgrade
- Security patch
- Database migration
- Server migration
- Framework update

ဒီအချိန်မှာ service downtime ဖြစ်နိုင်ပါတယ်။

```text
System Maintenance Window
12:00 AM - 3:00 AM
Service may be unavailable.
```

ဒါကြောင့် company တွေက maintenance window ကို ကြိုတင်စီစဉ်ရပါတယ်။

---

### 3.7 Active / Passive Setup ရဲ့ Complexity

Downtime လျှော့ချဖို့ တချို့ system တွေမှာ Active / Passive configuration သုံးပါတယ်။

```text
Active Server  → Currently serving users
Passive Server → Standby backup
```

Active server ပျက်သွားရင် Passive server ကို ပြောင်းသုံးနိုင်ပါတယ်။

ဒါပေမယ့် ပြဿနာအသစ်တွေရှိပါတယ်။

- Server နှစ်လုံးစလုံး patch level တူရမယ်
- Configuration တူရမယ်
- Data sync လုပ်ရမယ်
- License cost တိုးနိုင်တယ်
- System engineers တွေအတွက် maintain လုပ်ရတာ ပိုခက်လာတယ်

---

## 4. Monolith vs Microservices

| Topic | Monolith | Microservices |
|---|---|---|
| Structure | Application တစ်ခုတည်း | Service အသေးများခွဲထားသည် |
| Deployment | App တစ်ခုလုံး deploy | Service တစ်ခုချင်း deploy |
| Scaling | App တစ်ခုလုံး scale | Feature / service တစ်ခုချင်း scale |
| Failure Impact | App တစ်ခုလုံးထိခိုက်နိုင် | Service တစ်ခုသာထိခိုက်နိုင် |
| Maintenance | Codebase ကြီးလာရင်ခက် | Service ခွဲထားလို့ manage လုပ်လွယ်နိုင် |
| Cost | Hardware ကြီးလိုနိုင် | Resource ကိုလိုအပ်သလိုခွဲသုံးနိုင် |

---

## 5. Simple Analogy

### Monolith

Monolith ကို **အိမ်ကြီးတစ်လုံးထဲမှာ အခန်းတွေအကုန်တွဲထားတာ** လို့မြင်နိုင်ပါတယ်။

အခန်းတစ်ခန်းပြင်ချင်ရင်တောင် အိမ်တစ်လုံးလုံးကို ထိခိုက်နိုင်ပါတယ်။

### Microservices

Microservices ကို **အိမ်သေးသေးလေးတွေ သီးသန့်ခွဲထားတာ** လို့မြင်နိုင်ပါတယ်။

တစ်လုံးပျက်ရင် တစ်လုံးပဲပြင်လို့ရပြီး တစ်ခုချင်းစီကို သီးသန့် scale လုပ်လို့ရပါတယ်။

---

## 6. Main Idea Summary

Legacy Monolith application တွေကို cloud ပေါ်တင်ချင်တိုင်း lift-and-shift လုပ်လို့မလွယ်ပါဘူး။

အဓိက အကြောင်းရင်းတွေက:

1. Codebase ကြီးပြီး ရှုပ်တယ်
2. Architecture ဟောင်းတယ်
3. Programming language / framework ဟောင်းနိုင်တယ်
4. Build / compile / test ကြာတယ်
5. Hardware အများကြီးလိုတယ်
6. Feature တစ်ခုချင်းစီကို scale လုပ်မရဘူး
7. Application တစ်ခုလုံးကိုပဲ scale လုပ်ရတယ်
8. Upgrade / patch / migration လုပ်ရင် downtime ဖြစ်နိုင်တယ်
9. High availability setup လုပ်ရင် cost နဲ့ complexity တိုးတယ်

---

## 7. One-line Note

> **Legacy Monolith ဆိုတာ feature တွေအကုန်လုံး application တစ်ခုတည်းထဲမှာစုပြီးရေးထားတဲ့ legacy system ကြီးဖြစ်ပြီး cloud migration, scaling, maintenance, upgrade လုပ်ရာမှာ cost, downtime, complexity တွေများနိုင်တဲ့ architecture ဖြစ်ပါတယ်။**

---

## 8. Quick Revision Points

- Monolith = One big application
- Legacy = Old system / old architecture
- Cloud migration = Not always easy
- Scaling = Whole app ကိုပဲ scale လုပ်ရတယ်
- Hardware = Expensive and powerful server လိုနိုင်တယ်
- Downtime = Upgrade / patch လုပ်ရင် ဖြစ်နိုင်တယ်
- Active / Passive = Downtime လျှော့ပေမယ့် complexity တိုးတယ်
- Main problem = Cost + Complexity + Scalability + Downtime

