# Promise-pool
promise并发控制

## 用法
```js
// promise工厂数组
const promiseList = [() => fetch('https://www.apiopen.top/novelApi'), () => fetch('https://www.apiopen.top/novelApi'), () => fetch('https://www.apiopen.top/novelApi'), () => fetch('https://www.apiopen.top/novelApi'), () => fetch('https://www.apiopen.top/novelApi')]

// 向Promise对象添加pool方法，number为并发数
        Promise.pool = function (promiseGeneratorList, number) {
          return new Promise((resolve, reject) => {
            const result = []
            const len = promiseGeneratorList.length
            let status = 'pending' // pool的状态
            const thread = () => { // thread代表一个队列
              const promiseGenerator = promiseGeneratorList.shift()

              if (!promiseGenerator) { return }
              promiseGenerator()
                .then(res => {
                  result.push(res)

                  console.log(result.length, promiseGeneratorList.length)
                  if (result.length === len) {
                    status = 'fulfilled'
                    resolve(result) // 全部成功时触发
                  }

                  if (status === 'pending') {
                    thread()
                  }
                })
                .catch(error => {
                  status = 'rejected'
                  reject(error) // 任意一个失败时触发
                })
            }

            for (let i = 0; i < number; i++) { // 生成number个队列，达到最大并发数
              thread()
            }
          })
        }

        Promise.pool(promiseList, 3)
          .then(res => {
            console.log(res)
          })
          .catch(err => {
            console.log(err)
          })
```

## 说明
> 如果不增加status判断也可以实现所需功能，但是当某一个promise的状态reject时只会停止该thread，其它thread仍然会继续执行，虽然对结果无影响，但是没必要执行
