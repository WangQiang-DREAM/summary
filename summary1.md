### 简单背单词总结

#### mobx使用

1. store变量声明问题<br>
    mobx创建可观察对象的过程比较复杂，若创建的变量数据结构比较复杂，变量深层数据数据发生变化时会引起较为复杂的计算，可能会导致多次render，所以为了性能考虑，不建议创建复杂的可观察对象。此时用一个其他变量来监测数据是否发生变化。
    例如：<br>
    `store/js`
    ```
        @observable updateCount = 0;
        data = {
            code:0,
            message:'message',
            data:{
                page:1,
                total:100,
                data:[
                    ...
                ]
            }
        }

        @action getData = async () => {
            let result = await getData();
            this.data = result;
            this.updateCount++;
        }
    ```
    `component`
    ```  

        class App extends Commpoent{
            componentDidMount(){
                this.getData()
            }
            getData(){
                this.props.store.getData()
            }
            render(){
                let updateCount = this.props.store.updateCount;
                return(
                    ...
                )
            }
        }
    ```
    这样在data数据发生变化后，只走了一次updateCount的自增，我们通过观察updateCount的变化来重新render dom

2. store变量修改问题<br>
    store内的变量，为了规范，无论是声明和修改，都应该在store内进行，比如：<br>
    `store/js`
    ```
        @observable name = '哈哈哈';

        @action changeName(value){
            this.name = value;
        }
    ```
      `component`
    ```
        class App extends Commpoent{
            changeName(){
                 this.props.changeName('哈哈哈')
            }
        }
    ```
    不推荐以下写法：<br>
    ```
        class App extends Commpoent{
            changeName(){
                 this.props.store.name = '哈哈哈'
            }
        }
    ```
3. react中的滚动事件监听
    ```
        class App extends Commpoent{
            componentDidMount(){
                window.addEventListener('scroll',this.scroll); 
            }

            componentWillUnmount(){
                window.addEventListener('scroll',this.scroll);
           }
           
            scroll(){
               console.log('scroll');
            }
        } 
    ```
    不要在监听事件里写匿名函数，会导致组件销毁时监听事件无法移除
  
4. 父子组件的生命周期顺序和页面数据空白期的处理<br>
    当组件在componentDidMount的生命周期里请求大量初始化数据时，可能会导致长时间的空白，并且在render里可能会有找不到数据渲染而出错的问题，比如在父组件的ComponentDidMount里请求登录信息数据,在子组件的ComponentDidMount的利用登录信息查询，例如
    ```
    //父组件
    class Parent extends Compoenent{
        componentDidMount(){
            this.getUserInfo();
        }

        async getUserInfo(){
            await getUserInfo()
        }

        render(){
            return(
                <Children>
            )
        }
    }
    ```
    ```
    //子组件
    class Children extends Component{
        componentDidMount(){
            this.getData(userInfo)
        }

        async getData(){
            await getData(userInfo)
        }
    }
    ```
    `store`
    ```
        async getUserInfo(param){
            try {
                let result = await getUserInfo(param);
                dispatch(result);
            } catch(error){
                throw error
            }
        } 
    ```
    由于子组件的componentDidMount会在父组件的componentDidMount之前运行，所以在执行getData方法时会因为没有用户数据而出错，所以在父组件的userInfo信息拿到之前应该不能执行子组件的方法，换言之，也就是不渲染Children组件。
    在父组件的render方法里应该加以控制，给store做以下控制,定义一个可观察变量初始化isReady = false<br>
     `store`
    ```
        async getUserInfo(param){
            try {
                let result = await getUserInfo(param);
                dispatch(result);
            } catch(error){
                throw error
            } finally {
                this.isReady = true
            }
        } 
    ```
     ```
    //父组件
    class Parent extends Component{
        componentDidMount(){
            this.getUserInfo();
        }

        async getUserInfo(){
            await getUserInfo()
        }

        render(){
            if(this.props.store.isReady){
                return(
                    <Children>
                )
            }
        }
    }
    ```
5. 部分浏览器get请求走缓存的问题<br>
    ie11等部分浏览器在发送参数一样的get请求时，会默认走前一次的缓存，为了解决这个问题，可以在get请求的参数里多加一个时间戳参数，如timestamp = 14589902183,这样就能让每一次的get请求不一样
