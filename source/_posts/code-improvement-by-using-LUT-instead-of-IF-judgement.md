---
title: Code flow improvement by using the LUT instead of the if condition judgement
date: 2023-11-28 15:21:54
tags: ["前端"]
---

Today`s topic is using the LUT instead of the if condition judgement to improve the code flow.

Sometimes, there will be a huge if-else code block in our daily requirement implementation. like this:

```javascript
if(){
    if(){
        if(){
            if(){
                if(){
                    if(){
                        // ...condition
                    }else{
                        // ... condition
                    }
                }else{

                }
            }else{

            }
        }else{

        }
    }else{

    }
}else{

}
```

It will cause that a very high cognitive complexity, meanwhile, it will be very difficult to review the code block.

### There are some solution for that.

#### 1.Replace with switch statement

```javascript
const expr = "Papayas";
switch (expr) {
  case "Oranges":
    console.log("Oranges are $0.59 a pound.");
    break;
  case "Mangoes":
  case "Papayas":
    console.log("Mangoes and papayas are $2.79 a pound.");
    // Expected output: "Mangoes and papayas are $2.79 a pound."
    break;
  default:
    console.log(`Sorry, we are out of ${expr}.`);
}
```

#### 2.Replace with the LUT(lookup table)  
Actually, this is a space-for-time approach. Here is a real case:

```javascript
    {
      let a: string = "";
      if (!this.limitIsEmpty(v.LLL))
      {
        a = a + v.LLL;
      }
      if (!this.limitIsEmpty(v.LL))
      {
        a = a + v.LL;
      }
      if (!this.limitIsEmpty(v.L))
      {
        a = a + v.L;
      }
      if (!this.limitIsEmpty(v.H))
      {
        a = a + v.H;
      }
      if (!this.limitIsEmpty(v.HH))
      {
        a = a + v.HH;
      }
      if (!this.limitIsEmpty(v.HHH))
      {
        a = a + v.HHH;
      }
      return a;
    };

// we can using the LUT to improve the huge IF code block
    {
      const valueSet: string[] = ["LLL", "LL", "L", "H", "HH", "HHH"];
      let a: string = "";
      for (const item of valueSet)
      {
        const v_: string = v[item as keyof typeof v];
        if (!this.limitIsEmpty(v_))
        {
          a = a + v_;
        }
      }
      return a;
    };
```
