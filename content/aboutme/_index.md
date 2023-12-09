---
hidemeta: true
---

<style>

    @media screen and (max-width: 768px){
        .container{
            display:flex;
            flex-direction: column;
            justify-content: center;
            height:70vh;
        }
        .leftBox{
            width:100%;
            height:30vh;
            display: inline-block; 
            align-items: center; 
            display:flex;
            flex-direction: column;
            justify-self:flex-start;
        }
        .rightBox{
            width:100%;
            height:20vh;
            display: inline-block;
            align-items: center; 
            display:flex;
            flex-direction: column;
            justify-content: flex-start;
        }
        .introBox{
            font-size: 12px;
        }
        .introBox > span{
            font-weight: bold;
            font-size: 18px; 
            color: black;
        }
    }
    @media screen and (min-width: 768px){
        .container{
            display:flex;
            flex-direction: column;
            justify-content: center;
        }
        .leftBox{
            width:100%;
            height:40vh;
            display: inline-block; 
            align-items: center; 
            display:flex;
            flex-direction: column;
            justify-content: flex-start;
        }
        .rightBox{
            width:100%;
            height:40vh;
            display: inline-block;
            align-items: center; 
            display:flex;
            flex-direction: column;
            justify-content: flex-start;
        }
        .introBox{
            font-size: 14px;
        }
        .introBox > span{
            font-weight: bold;
            font-size: 24px; 
            color: black;
        }
    }
    </style>
    
<div class="container">
            <div class="leftBox">
                  <img src="https://i.imgs.ovh/2023/11/12/nLRSp.md.png" width=200 height=200/>
            </div>
            <div class="rightBox">
                <center class="introBox">
                <span>SwimmingLiu 👨🏻‍🎓</span> <br/>
                Master in  <span>ZSTU</span>, Majoring <span>Computer Science 💻</span> <br/>
                Exploring the <span>World</span> with <span>Computer Vision 🌎</span> 
                </center>
            </div>
</div>
    
<script>
    
    
        // 监听 body 元素的 classList 变化
    const bodyObserver = new MutationObserver(mutations => {
      mutations.forEach(mutation => {
        if (mutation.type === 'attributes' && mutation.attributeName === 'class') {
          const bodyClass = document.body.classList;
          // 检查 body 的 class 是否包含特定的类名
    
          if (bodyClass.contains('dark')) {
            // 修改 introBox 类中的字体颜色
            const introBox = document.querySelector('.introBox');
            if (introBox) {
                const spanElements = introBox.querySelectorAll('span');
                if (spanElements) {
                    // 修改所有 span 元素的颜色
                    spanElements.forEach(spanElement => {
                    spanElement.style.color = 'white';
                    });
                }
            }
          }
          else{
             // 修改 introBox 类中的字体颜色
             const introBox = document.querySelector('.introBox');
            if (introBox) {
                    const spanElements = introBox.querySelectorAll('span');
                    if (spanElements) {
                        // 修改所有 span 元素的颜色
                        spanElements.forEach(spanElement => {
                        spanElement.style.color = 'black';
                        });
                    }
                }
                }
          }
        }
      );
    });
    
    // 开始观察 body 元素的 classList 变化
    bodyObserver.observe(document.body, { attributes: true });
    
</script>