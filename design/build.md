为什么要学建造者模式模式
> - 什么是建造者模式
> - 建造者模式使用场景
> - 如何实现一个建造者模式
> - 建造者模式优点和缺点

## 什么是建造者模式

建造者模式（Builder Pattern）

由多个简单对象一步一步构建成为一个复杂对象，属于创建型工厂模式

通过一个builder类来一步步实现搭建复杂对象


## 建造者模式使用场景

**1、需要生成的对象具有复杂的内部结构**

**2、需要生成的对象内部属性本身相互依赖。**


## 如何实现一个建造者模式

```java

/**
 * @author jiangtao
 * @version V1.0
 * @Description：支付统一响应报文 <p>创建日期：2019/6/10 </p>
 * @see
 */
public class PayResponseBody {

    //功能码
    private String funCode;
    
    //流水号
    private String rpid;
    
    //商户id
    private String merId;
    
    //交易金额
    private String amount;
    
    //响应日期
    private String resdate;
    
    //响应时间
    private String restime;
    
    //银行卡号
    private String account;
    
    //银行名称
    private String bankName;
    
    //银行信息
    private String bankInfo;

    public String getFunCode() {
        return funCode;
    }

    public void setFunCode(String funCode) {
        this.funCode = funCode;
    }

    public String getRpid() {
        return rpid;
    }

    public void setRpid(String rpid) {
        this.rpid = rpid;
    }

    public String getMerId() {
        return merId;
    }

    public void setMerId(String merId) {
        this.merId = merId;
    }

    public String getAmount() {
        return amount;
    }

    public void setAmount(String amount) {
        this.amount = amount;
    }

    public String getResdate() {
        return resdate;
    }

    public void setResdate(String resdate) {
        this.resdate = resdate;
    }

    public String getRestime() {
        return restime;
    }

    public void setRestime(String restime) {
        this.restime = restime;
    }

    public String getAccount() {
        return account;
    }

    public void setAccount(String account) {
        this.account = account;
    }

    public String getBankName() {
        return bankName;
    }

    public void setBankName(String bankName) {
        this.bankName = bankName;
    }

    public String getBankInfo() {
        return bankInfo;
    }

    public void setBankInfo(String bankInfo) {
        this.bankInfo = bankInfo;
    }

    @Override
    public String toString() {
        return "PayResponseBody{" +
                "funCode='" + funCode + '\'' +
                ", rpid='" + rpid + '\'' +
                ", merId='" + merId + '\'' +
                ", amount='" + amount + '\'' +
                ", resdate='" + resdate + '\'' +
                ", restime='" + restime + '\'' +
                ", account='" + account + '\'' +
                ", bankName='" + bankName + '\'' +
                ", bankInfo='" + bankInfo + '\'' +
                '}';
    }
}

/**
 * @author jiangtao
 * @version V1.0
 * @Description：抽象响应构建工厂 <p>创建日期：2019/6/10 </p>
 * @see
 */
public abstract class PayResponseHandler {

    public abstract void  funCode(String funCode);
    
    public abstract void  rpid(String rpid);
    
    public abstract void  merId(String merId);
    
    public abstract void  amount(String amount);
    
    public abstract void  resdate(String resdate);
    
    public abstract void  restime(String restime);
    
    public abstract void  account(String account);
    
    public void bankName(String bankName){}
    
    public void bankInfo(String bankInfo){}
    
    //创建响应对象
    public abstract PayResponseBody creat();
}

/**
 * @author jiangtao
 * @version V1.0
 * @Description：微信响应对象 <p>创建日期：2019/6/10 </p>
 * @see
 */
public class BankResponseBody extends PayResponseHandler {

    private PayResponseBody payResponseBody = new PayResponseBody();

    @Override
    public void funCode(String funCode) {
        payResponseBody.setFunCode(funCode);
    }

    @Override
    public void rpid(String rpid) {
        payResponseBody.setRpid(rpid);
    }

    @Override
    public void merId(String merId) {
        payResponseBody.setMerId(merId);
    }

    @Override
    public void amount(String amount) {
        payResponseBody.setAmount(amount);
    }

    @Override
    public void resdate(String resdate) {
        payResponseBody.setResdate(resdate);
    }

    @Override
    public void restime(String restime) {
        payResponseBody.setRestime(restime);
    }

    @Override
    public void account(String account) {
        payResponseBody.setAccount(account);
    }
    
    public  void  bankName(String bankName){
        payResponseBody.setBankName(bankName);
    }

    public  void  bankInfo(String bankInfo){
        payResponseBody.setBankInfo(bankInfo);
    }

    @Override
    public PayResponseBody creat() {
        return payResponseBody;
    }

    @Override
    public String toString() {
        return "payResponseBody=" + payResponseBody ;
    }
}

/**
 * @author jiangtao
 * @version V1.0
 * @Description：微信响应对象 <p>创建日期：2019/6/10 </p>
 * @see
 */
public class WxResponseBody extends PayResponseHandler {

    private PayResponseBody payResponseBody = new PayResponseBody();

    @Override
    public void funCode(String funCode) {
        payResponseBody.setFunCode(funCode);
    }

    @Override
    public void rpid(String rpid) {
        payResponseBody.setRpid(rpid);
    }

    @Override
    public void merId(String merId) {
        payResponseBody.setMerId(merId);
    }

    @Override
    public void amount(String amount) {
        payResponseBody.setAmount(amount);
    }

    @Override
    public void resdate(String resdate) {
        payResponseBody.setResdate(resdate);
    }

    @Override
    public void restime(String restime) {
        payResponseBody.setRestime(restime);
    }

    @Override
    public void account(String account) {
        payResponseBody.setAccount(account);
    }

    @Override
    public PayResponseBody creat() {
        return payResponseBody;
    }

    @Override
    public String toString() {
        return "payResponseBody=" + payResponseBody ;
    }
}


/**
 * @author jiangtao
 * @version V1.0
 * @Description：微信响应对象 <p>创建日期：2019/6/10 </p>
 * @see
 */
public class WxResponseBodyBuilder {

    private PayResponseHandler payResponseHandler;

    public WxResponseBodyBuilder(PayResponseHandler payResponseHandler){
        this.payResponseHandler =  payResponseHandler;
    }

    //执行构造参数
    public PayResponseBody constuct(String account,String amount,String funCode,String merId,String resdate,String restime,String rpid){
        
        payResponseHandler.account(account);
        
        payResponseHandler.amount(amount);
        
        payResponseHandler.funCode(funCode);
        
        payResponseHandler.merId(merId);
        
        payResponseHandler.resdate(resdate);
        
        payResponseHandler.restime(restime);
        
        payResponseHandler.rpid(rpid);
        
        return  payResponseHandler.creat();
    }


}

/**
 * @author jiangtao
 * @version V1.0
 * @Description：微信响应对象 <p>创建日期：2019/6/10 </p>
 * @see
 */
public class BankResponseBodyBuilder{

    private PayResponseHandler payResponseHandler;

    public BankResponseBodyBuilder(PayResponseHandler payResponseHandler){
        this.payResponseHandler =  payResponseHandler;
    }

    //执行构造参数
    public PayResponseBody constuct(String account,String amount,String bankInfo,String bankName,String funCode,String merId,String resdate,String restime,String rpid){
       
        payResponseHandler.account(account);
       
        payResponseHandler.amount(amount);
       
        payResponseHandler.bankInfo(bankInfo);
       
        payResponseHandler.bankName(bankName);
       
        payResponseHandler.funCode(funCode);
       
        payResponseHandler.merId(merId);
       
        payResponseHandler.resdate(resdate);
       
        payResponseHandler.restime(restime);
       
        payResponseHandler.rpid(rpid);
       
        return  payResponseHandler.creat();
    }
    
}

/**
 * @author jiangtao
 * @version V1.0
 * @Description： <p>创建日期：2019/6/11 </p>
 * @see
 */
public class ResponseBuilderTest {
    
    public static void main(String[] args) {
    
        //创建银行卡支付后结果响应报文
        BankResponseBody bankResponseBody = new BankResponseBody();
        
        BankResponseBodyBuilder bankResponseBodyBuilder = new BankResponseBodyBuilder(bankResponseBody);
        
        bankResponseBodyBuilder.constuct("6226090108554561", "1", "招商银行支付", "招商银行", "BankPay", "9996", "20190501", "164004", "123456789");
       
        System.out.println("银行卡响应报文："+bankResponseBody.toString());
        
        //创建微信支付完成后结果响应报文
        WxResponseBody wxResponseBody = new WxResponseBody();
       
        WxResponseBodyBuilder wxResponseBodyBuilder = new WxResponseBodyBuilder(wxResponseBody);
        
        wxResponseBodyBuilder.constuct("123465798555","1","WXpay","9996","20190601","080101","132456789456");
        
        System.out.println("微信响应报文："+wxResponseBody.toString());
    }
}
```


这段代码的输出如下：
```

银行卡响应报文：

{funCode='BankPay', rpid='123456789', merId='9996', amount='1', resdate='20190501', restime='164004', account='6226090108554561', bankName='招商银行', bankInfo='招商银行支付'}

微信响应报文：

{funCode='WXpay', rpid='132456789456', merId='9996', amount='1', resdate='20190601', restime='080101', account='123465798555', bankName='null', bankInfo='null'}



```

## 建造者模式的优点与缺点
优点
>- 建造者独立，易扩展。
>- 便于控制细节风险。 

缺点
>- 产品必须有共同点，范围有限制
>- 如内部变化复杂，会有很多的建造类

## 扩展建造者的经典模式
```
package com.example.factory_demo.builder.kfc;

/**
 * @author jiangtao
 * @version V1.0
 * @Description：商品列表 <p>创建日期：2019/6/12 </p>
 * @see
 */
public interface ProductItem {

    /**
     * 商品列表
     * @return
     */
    public String productName();

    /**
     * 商品包装
     */
    public ProductPackage packageing();

    /**
     * 商品价格
     * @return
     */
    public float price();
}

package com.example.factory_demo.builder.kfc;

/**
 * 打包方式：纸袋包装，水瓶包装
 */
public interface ProductPackage {

    public String packages();
}

package com.example.factory_demo.builder.kfc;

/**
 * @author jiangtao
 * @version V1.0
 * @Description：水瓶包装 <p>创建日期：2019/6/12 </p>
 * @see
 */
public class BottlePackage implements ProductPackage {
    @Override
    public String packages() {
        return "水瓶包装";
    }
}

package com.example.factory_demo.builder.kfc;

/**
 * @author jiangtao
 * @version V1.0
 * @Description：纸袋包装 <p>创建日期：2019/6/12 </p>
 * @see
 */
public class WapperPackage implements ProductPackage {
    @Override
    public String packages() {
        return "纸袋包装";
    }
}


package com.example.factory_demo.builder.kfc;

/**
 * @author jiangtao
 * @version V1.0
 * @Description：汉堡的抽象类 <p>创建日期：2019/6/12 </p>
 * @see
 */
public abstract class Burger implements ProductItem{

    /**
     * 定义包含的包装是什么
     * @return
     */
    @Override
    public ProductPackage packageing(){
        return new WapperPackage();
    }

    /**
     * 抽象定义汉堡的价格
     * @return
     */
    public abstract float price();
}

package com.example.factory_demo.builder.kfc;


/**
 * @author jiangtao
 * @version V1.0
 * @Description：定义饮品的包装方式 <p>创建日期：2019/6/12 </p>
 * @see
 */
public abstract class ColdDrink implements ProductItem{

    /**
     * 定义饮品的包装方式统一为能装水的杯子
     * @return
     */
    @Override
    public ProductPackage packageing(){
        return new BottlePackage();
    }

    /**
     * 定义饮品的价格
     * @return
     */
    public abstract float price();
}

package com.example.factory_demo.builder.kfc;

/**
 * @author jiangtao
 * @version V1.0
 * @Description：肉食汉堡 <p>创建日期：2019/6/12 </p>
 * @see
 */
public class BeefBurger extends Burger{
    @Override
    public String productName() {
        return "香辣鸡腿堡";
    }

    @Override
    public float price() {
        return 25.0f;
    }
}

package com.example.factory_demo.builder.kfc;

/**
 * @author jiangtao
 * @version V1.0
 * @Description：素食汉堡 <p>创建日期：2019/6/12 </p>
 * @see
 */
public class VegetableBurger extends Burger {
    @Override
    public String productName() {
        return "无肉汉堡(友情提示不吃，但是几个实惠)";
    }

    @Override
    public float price() {
        return 15.0f;
    }
}

package com.example.factory_demo.builder.kfc;

/**
 * @author jiangtao
 * @version V1.0
 * @Description：可口可乐的包装和价格 <p>创建日期：2019/6/12 </p>
 * @see
 */
public class Coke extends ColdDrink {
    @Override
    public String productName() {
        return "可口可乐.店长推荐，你指的拥有";
    }

    @Override
    public float price() {
        return 11.0f;
    }
}

package com.example.factory_demo.builder.kfc;

/**
 * @author jiangtao
 * @version V1.0
 * @Description：百事可乐的包装和价格 <p>创建日期：2019/6/12 </p>
 * @see
 */
public class Pepsi extends ColdDrink {
    @Override
    public String productName() {
        return "百事可乐.本人不喜欢";
    }

    @Override
    public float price() {
        return 11.0f;
    }
}

package com.example.factory_demo.builder.kfc;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicReference;

/**
 * @author jiangtao
 * @version V1.0
 * @Description：购买的尚品列表 <p>创建日期：2019/6/12 </p>
 * @see
 */
public class ProductItems {
    //本次购买的商品列表
    private List<ProductItem> productItemList = new ArrayList<>();

    /**
     * 添加购买的商品
     * @param productItem
     */
    public  void add(ProductItem productItem){
        productItemList.add(productItem);
    }

    /**
     * 获取商品的总价格
     * @return
     */
    public float totalPrice(){
        AtomicReference<Float> prices = new AtomicReference<>(0f);
        productItemList.forEach((ProductItem productItem)->{
            prices.updateAndGet(v -> new Float((float) (v + productItem.price())));
        });
        return prices.get();
    }

    /**
     * 打印购买结果
     */
    public void print(){
        productItemList.forEach((ProductItem productItem)->{
            System.out.print("item name:"+productItem.productName());
            System.out.print(" , package:"+productItem.packageing().packages());
            System.out.println(", price:"+productItem.price());
        });
    }
}

package com.example.factory_demo.builder.kfc;

/**
 * @author jiangtao
 * @version V1.0
 * @Description：构建商品套餐 <p>创建日期：2019/6/12 </p>
 * @see
 */
public class ProductItemsBuilder {

    /**
     * 购买香辣鸡腿堡的可乐套餐
     * @return
     */
    public ProductItems beefAndCokeItems(){
        ProductItems productItems = new ProductItems();
        productItems.add(new BeefBurger());
        productItems.add(new Coke());
        return productItems;
    }

    /**
     * 购买素食汉堡和可乐套餐
     * @return
     */
   public ProductItems vegetableAndCokeItemd(){
        ProductItems productItems = new ProductItems();
        productItems.add(new VegetableBurger());
        productItems.add(new Coke());
        return productItems;
   }
}

package com.example.factory_demo.builder.kfc;

/**
 * @author jiangtao
 * @version V1.0
 * @Description：jt的中午选择不同的套餐，需要花费的金额是多少 <p>创建日期：2019/6/12 </p>
 * @see
 */
public class JtSkyDemo {

    public static void main(String[] args) {
        System.out.println("本人想吃一个肉汉堡+可乐，计算一下花费金额");
        ProductItemsBuilder productItemsBuilder = new ProductItemsBuilder();
        ProductItems productItems = productItemsBuilder.beefAndCokeItems();
        productItems.print();
        System.out.println("本次购买的肉汉堡花费="+productItems.totalPrice());
        System.out.println();
        System.out.println("本人想吃一个素食汉堡+可乐，计算一下花费金额");
        ProductItemsBuilder productItemsBuilder1 = new ProductItemsBuilder();
        ProductItems productItems1 = productItemsBuilder1.vegetableAndCokeItemd();
        productItems1.print();
        System.out.println("本人购买的素汉堡="+productItems1.totalPrice());
    }
}

```
根据自己中午的意愿随意选择要吃的商品，最终会给输出一个购买商品的列表和你要花费的金额

```
本人想吃一个肉汉堡+可乐，计算一下花费金额
item name:香辣鸡腿堡 , package:纸袋包装, price:25.0
item name:可口可乐.店长推荐，你指的拥有 , package:水瓶包装, price:11.0
本次购买的肉汉堡花费=36.0

本人想吃一个素食汉堡+可乐，计算一下花费金额
item name:无肉汉堡(友情提示不吃，但是几个实惠) , package:纸袋包装, price:15.0
item name:可口可乐.店长推荐，你指的拥有 , package:水瓶包装, price:11.0
本人购买的素汉堡=26.0

```