# @RequestBody

~~~java
/**
 * 通过项目详细信息id修改项目详细信息
 * @param {*} param 
 * @returns 
 */
export function projectDeatailInfoUpdate(param){
  return http({
    url: "/api/affairs/returnCultivatedLandToField/updateDatialDateById",
    data: param,  
    method: "post",
    headers: {
      "Content-Type": "application/json",
    },
  });
}

~~~

headers: { "Content-Type": "application/json",},

加了这个代码就意味着我们的请求体要以json格式发送， 然后我们的@RequestBody就是来接受JSON格式的消息体.