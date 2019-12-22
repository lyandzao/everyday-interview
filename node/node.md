## 1.使用NodeJS编写代码实现遍历文件夹及所有文件名

```javascript
const fs=require('fs')
const path = require('path')

const readDir = (entry) => {
  const dirInfo = fs.readdirSync(entry)
  dirInfo.forEach(item => {
    const location = path.join(entry, item)
    const info=fs.statSync(location)
    if (info.isDirectory()) {
      console.log(`dir:${location}`)
      readDir(location)
    } else {
      console.log(`file:${location}`)
      
    }
  })
}

readDir(__dirname)
```

## 2. 