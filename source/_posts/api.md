---
category: util
title: Api文档自动生成的调研
date: 2017-05-17 10:52:19
tags: 工具
toc: true
---

## 现状
  目前团队采用前后端分离的开发模式，需要后端人员提供接口文档，目前的现状后端人员先把接口的逻辑代码写的差不多再以word接口文档的方式交付给前端，存在问题：
         
>  -  接口改动时，不易被识别
>  -  不能进行测试
>  -  需要单独维护接口文档
>  -  文档更新不及时，更新不方便
>  -  前端需要依赖接口数据进行开发
>  -  代码和文档分离，不好维护

## 背景

   前后端分离已经是业界所共识的一种开发/部署模式了，
   后端负责数据的提供和计算，前端负责数据的展现和交互逻辑，
   前后端通过RESTFul的接口来通信，解耦前后端的开发过程，细化职责，提高了开发效率。
   但是前后端集成的效率成为一个绕不开的问题，由于前后端并行开发，在集成联调时需要花费大量的精力和时间来调试。
   并且随着项目的发展，异构网络及微服务的引入，后端会越来越复杂，一个Ａｐｉ接口可能需要若干个成员的共同维护，在这样复杂的系统中，后台任意端点的失败都可能影响前端集成开发的流程。
   一般会采用mock的方式来解决这个问题,所谓mock服务，就是对某些不容易构造或者不容易获取的对象，创建一个虚拟的对象以便测试开发。
   ![image](http://ocpue1vvp.bkt.clouddn.com/Mock.jpeg)
   Mock服务解决了前端依赖接口数据进行开发的问题，进一步解耦前后端，做到了接口设计先行，定下接口契约后各端并行开发。
   并将这些契约作为可以被测试的中间格式。然后前后端都需要有测试来使用这些契约。一旦契约发生变化，则另一方的测试会失败，这样就会驱动双方协商，并提高集成时的效率。
   
   


## 目标
  API文档管理+API测试+Mock+用的爽！
  
## 选型
  有了以上目标，接下来的就是进行工具选型。
- RAP
   
RAP是一个可视化接口管理工具 通过分析接口结构，动态生成模拟数据，校验真实接口正确性， 围绕接口定义，通过一系列自动化工具提升我们的协作效率。

![image](http://ocpue1vvp.bkt.clouddn.com/RAP.png)
- ApiDoc

apidoc 是一个简单的API 文档生成工具，它从代码注释中提取特定格式的内容，生成文档。相对而言，web接口的注释维护起来更加方便，不需要额外再维护一份文档。

apidoc 拥有以下特点：

1.  跨平台，linux、windows、macOS 等都支持；
2. 支持语言广泛，即使是不支持，也很方便扩展；
3. 支持多个不同语言的多个项目生成一份文档；
4. 输出模板可自定义；

```
/**
	* @api {post} /admin/admin/adminGet 工作人员详情
	* @apiDescription 获取工作人员详情
	* @apiName adminGet
	* @apiGroup admin
	* @apiVersion 1.0.0
	* 
	* @apiParam {Number} adminId 工作人员ID
	* @apiParamExample {json} Request Example
 	{
    "requestHeader": {
        "apiName": "adminGet",
        "version": "v1.0.0",
        "source": "saas",
        "token": "gym_login_token_dd36cd89-e27d-4a01-bdf2-eb712c395436"
    },
    "requestBody": {
        "adminId": "ACFJNOJIV"
     }
  }
	*
	* 
	* @apiSuccess {Number} code 响应码.
	* @apiSuccess {String} msg 提示
	* @apiSuccess {Number} subCode 提示码.
	* @apiSuccess {String} subMsg 提示信息
	* @apiSuccess {Json} response 结果
	* @apiSuccess (response) {Json} admin 人员信息
	* @apiSuccess (response) {Json} roleList 角色列表
	* 
	* @apiSuccess (admin) {String} adminId 工作人员ID
	* @apiSuccess (admin) {String} name 名字
	* @apiSuccess (admin) {String} phone 电话
	* @apiSuccess (admin) {String} level 级别 0超管 1普通
	* @apiSuccess (admin) {String} avatar 头像
	* @apiSuccess (admin) {Number} sex 性别
    * @apiSuccess (admin) {String} idCard 身份证
    * @apiSuccess (admin) {String} jobNumber 工号
    * @apiSuccess (admin) {String} birthday 出生日期
    * @apiSuccess (admin) {String} remark 备注
    * @apiSuccess (admin) {String} joinTime  加入时间
    * @apiSuccess (admin) {String} description 个人简介
    * @apiSuccess (admin) {String} specialty 特长
    * @apiSuccess (admin) {String} height 身高
    * @apiSuccess (admin) {Double} weight 体重
    * @apiSuccess (admin) {Double} workYear 工作年限
    * @apiSuccess (admin) {String} certificate 资质
    * @apiSuccess (admin) {String} motto 格言
    * @apiSuccess (admin)  {JSONArray} coachPicList 教练图片
    * 
	*
	* @apiSuccessExample {json} 返回样例:
	*
	*                    {
    "code": 0,
    "msg": "success",
    "subCode": "",
    "subMsg": "",
    "response": {
        "admin": {
            "adminId": "ACFJNOJIV",
            "avatar": "",
            "idCard": "123sedfe",
            "jobNumber": "ssd1",
            "joinTime": "2017-03-26",
            "level": 1,
            "name": "张教练",
            "phone": "18009090909",
            "remark": "这是员工",
            "sex": 1,
            "height": 185,
            "weight": 140.5,
            "workYear": 10,
            "certificate": "资质",
            "specialty": "擅长",
            "description": "个人简介",
            "birthday": "2010-01-01",
            "motto": "人生格言",
            "coachPicList": [
                {
                    "id": "ASDCDEDE6",
                    "url": ""
                }
            ]
        },
        "roleList": [
            "ABOMNB0B3"
        ]
    }
}
	*/

```

![image](http://ocpue1vvp.bkt.clouddn.com/apidoc1.png)


缺点：
   提供的测试功能过于简单
   key value形式的参数
   
- Swagger



> Swagger UI+Swagger Editor+Swagger API Spec

> Swagger UI+Swagger API Spec+SpringMVC



![image](http://ocpue1vvp.bkt.clouddn.com/swagger.png)