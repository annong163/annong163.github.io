---
title: Springboot深度整合xxl-job
author: AnNong
categories: xxl-job
img: /img/xxl-job.png
summary: XXL-JOB 是一个轻量级分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。
tags:
  - xxl-job
abbrlink: a18695c5
date: 2022-06-10 16:48:32
---

## 前言
XXL-JOB 是一个轻量级分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。
## 深度整合思路
内网搭建xxl-job-admin，通过接口形式调取新增、启动、停止、实时日志等方法实现项目内分布式大批量任务调度。
## 架构图
![](/img/xxl-架构.png)

## 搭建xxl-job-admin
### 1.拉取代码
[https://gitee.com/xuxueli0323/xxl-job.git](https://gitee.com/xuxueli0323/xxl-job.git)
### 2.代码结构
![](/img/xxl-admin结构.png)
### 3.运行sql文件
![](/img/xxl-sql.png)
### 4.修改 xxl-job-admin 模块的 yml 文件
![](/img/xxl-yml.png)
### 5.启动任务调度中心
> 账号：admin 密码：123456 （初始状态下）。
![](/img/xxl-login.png)

## SpringBoot整合
### 1. 引入依赖
```javascript
  <!-- xxlJob-job-core -->
        <dependency>
            <groupId>com.xuxueli</groupId>
            <artifactId>xxl-job-core</artifactId>
            <version>2.3.0</version>
        </dependency>
```
> 注意：此处版本要与 xxl-job-admin 中版本保持一致。

### 2. 项目整体引入结构
![](/img/project.jpg)

### 3. 新增XxlJobConfig类
```javascript
import com.xxl.job.core.executor.impl.XxlJobSpringExecutor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * xxlJob-job配置类
 *
 * @author Ma·JJ
 * @date 2021-07-26
 */
@Configuration
@Slf4j
public class XxlJobConfig {

    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;

    @Value("${xxl.job.accessToken}")
    private String accessToken;

    @Value("${xxl.job.executor.appname}")
    private String appname;

    @Value("${xxl.job.executor.address}")
    private String address;

    @Value("${xxl.job.executor.ip}")
    private String ip;

    @Value("${xxl.job.executor.port}")
    private int port;

    @Value("${xxl.job.executor.logpath}")
    private String logPath;

    @Value("${xxl.job.executor.logretentiondays}")
    private int logRetentionDays;


    @Bean
    public XxlJobSpringExecutor xxlJobExecutor() {
        log.info(">>>>>>>>>>> xxlJob-job config init.");
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppname(appname);
        xxlJobSpringExecutor.setAddress(address);
        xxlJobSpringExecutor.setIp(ip);
        xxlJobSpringExecutor.setPort(port);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);

        return xxlJobSpringExecutor;
    }


}

```
### 4. 修改application-dev.yml
```javascript
xxl:
  job:
    accessToken: Xz$jVwBJvKxcHiaptXW25H6^*HITxu!A
    jobGroupId: 5
    admin:
      addresses: http://125.74.198.209:8089/xxl-job-admin
    executor:
      address: ''
      appname: gkt-admin-system
      ip: ''
      logpath: /data/applogs/xxl-job/jobhandler
      logretentiondays: 30
      port: 6666
      collectHandler: collectHandler
      exchangeHandler: exchangeHandler
      analyseHandler: analyseHandler
```
### 5. 编写XxlJobController 用于项目前端调取xxl-job调度中心相关接口
```java

import com.xxl.job.core.biz.ExecutorBiz;
import com.xxl.job.core.biz.model.LogParam;
import com.xxl.job.core.biz.model.LogResult;
import com.xxl.job.core.biz.model.ReturnT;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.jeecg.common.api.vo.Result;
import org.jeecg.modules.xxlJob.service.XxlJobService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

/**
 * 定时任务统一处理中心
 *
 * @author Ma·JJ
 * @date 2021-07-27
 */
@RequestMapping("/xxlJob")
@Slf4j
@Api(tags = "XxlJob接口")
@RestController
public class XxlJobController {

    @Autowired
    private XxlJobService jobService;

    /**
     * 根据类型执行一次定时任务
     *
     * @param id     定时任务id
     * @param dataId 数据id
     * @param type   类型: 1 数据采集 2 数据交换 3 数据分析
     * @return Result
     */
    @GetMapping("/executeByDataId")
    public Result<?> executeByDataId(@RequestParam(name = "id", required = true) String id,
    @RequestParam(name = "dataId", required = true) String dataId,
    @RequestParam(name = "type", required = true) String type) {
    return jobService.executeByDataId(id, dataId, type);
}


/**
 * 根据类型启动定时任务
 *
 * @param id     定时任务id
 * @param dataId 数据id
 * @param type   类型: 1 数据采集 2 数据交换 3 数据分析
 * @return Result
 */
@GetMapping(value = "/resumeByDataId")
@ApiOperation(value = "启动定时任务")
public Result<Object> resumeByDataId(@RequestParam(name = "id", required = true) String id,
@RequestParam(name = "dataId", required = true) String dataId,
@RequestParam(name = "type", required = true) String type) {
    return jobService.resumeByDataId(id, dataId, type);
}

/**
 * 根据类型停止定时任务
 *
 * @param id     定时任务id
 * @param dataId 数据id
 * @param type   类型: 1 数据采集 2 数据交换 3 数据分析
 * @return Result
 */
@GetMapping(value = "/pauseByDataId")
@ApiOperation(value = "停止定时任务")
public Result<Object> pauseByDataId(@RequestParam(name = "id", required = true) String id,
@RequestParam(name = "dataId", required = true) String dataId,
@RequestParam(name = "type", required = true) String type,
@RequestParam(name = "flag", required = false) String flag) {
    return jobService.pauseByDataId(id, dataId, type, flag);
}

@GetMapping("/getCurrentLogInfo")
@ApiOperation("获取实时日志信息")
public Result<?> getCurrentLogInfo(@RequestParam("jobId") String jobId) {
    return jobService.getCurrentLogInfo(jobId);
}

@GetMapping("/logDetailCat")
@ApiOperation("获取实时日志详情")
public Result<?> logDetailCat(@RequestParam("executorAddress") String executorAddress,
@RequestParam("triggerTime") long triggerTime,
@RequestParam("logId") long logId,
@RequestParam("fromLineNum") int fromLineNum) {
    return jobService.logDetailCat(executorAddress, triggerTime, logId, fromLineNum);
}

/**
 * 批量同步数据至xxlJob
 *
 * @param ids
 * @return
 */
@ApiOperation("批量同步数据至xxlJob")
@GetMapping("/batchSync")
public Result<?> batchSync(@RequestParam(name = "ids", required = true) String ids,
@RequestParam(name = "type", required = true) String type) {
    return jobService.batchSync(ids, type);
}
}

```
### 6. 编写XxlJobServiceImpl 用于远程调取xxl-job调度中心接口
```javascript

import com.alibaba.fastjson.JSONObject;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang.StringUtils;
import org.jeecg.common.api.vo.Result;
import org.jeecg.common.constant.BusinessConstant;
import org.jeecg.common.constant.CommonConstant;
import org.jeecg.modules.system.entity.DataAnalyse;
import org.jeecg.modules.system.entity.DataCollect;
import org.jeecg.modules.system.entity.DataExchange;
import org.jeecg.modules.system.service.DataAnalyseService;
import org.jeecg.modules.system.service.DataCollectService;
import org.jeecg.modules.system.service.DataExchangeService;
import org.jeecg.modules.xxlJob.service.XxlJobService;
import org.jeecg.modules.xxlJob.utils.DispatchUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import java.util.Arrays;
import java.util.List;

/**
 * 定时任务统一处理中心
 *
 * @author Ma·JJ
 * @date 2021-07-27
 */
@Service
@Slf4j
public class XxlJobServiceImpl implements XxlJobService {

    @Autowired
    private DataCollectService dataCollectService;
    @Autowired
    private DataExchangeService dataExchangeService;
    @Autowired
    private DataAnalyseService dataAnalyseService;
    /**
     * 调度中心@XxlJob名称
     */
    @Value("${xxl.job.executor.collectHandler}")
    private String collectHandler;
    /**
     * 调度中心@XxlJob名称
     */
    @Value("${xxl.job.executor.analyseHandler}")
    private String analyseHandler;
    /**
     * 调度中心@XxlJob名称
     */
    @Value("${xxl.job.executor.exchangeHandler}")
    private String exchangeHandler;


    /**
     * 立即执行业务功能
     *
     * @param jobId  定时任务id
     * @param dataId 业务数据id
     * @param type   1: 数据采集 2: 数据交换 3 数据分析
     * @return Result
     */
    @Override
    public Result<?> executeByDataId(String jobId, String dataId, String type) {
        try {
            switch (type) {
                case BusinessConstant.TYPE_COLLECT:
                    DataCollect dataCollect = dataCollectService.getById(dataId);
                    dataCollect.setExecuteStatus(BusinessConstant.STATUS_2);
                    dataCollectService.updateById(dataCollect);
                    break;
                case BusinessConstant.TYPE_EXCHANGE:
                    DataExchange dataExchange = dataExchangeService.getById(dataId);
                    dataExchange.setExecuteStatus(BusinessConstant.STATUS_2);
                    dataExchangeService.updateById(dataExchange);
                    break;
                case BusinessConstant.TYPE_ANALYSE:
                    DataAnalyse dataAnalyse = dataAnalyseService.getById(dataId);
                    dataAnalyse.setExecuteStatus(BusinessConstant.STATUS_2);
                    dataAnalyseService.updateById(dataAnalyse);
                    break;
                default:
                    return Result.error("未匹配到类型");
            }

            JSONObject jsonObject = DispatchUtils.execute(jobId, dataId, BusinessConstant.JOB_TRIGGER);
            Integer code = jsonObject.getInteger(BusinessConstant.RESULT_CODE);
            String content = jsonObject.getString(BusinessConstant.RESULT_CONTENT);
            if (CommonConstant.SC_INTERNAL_SERVER_ERROR_500.equals(code)) {
                // 调度中心出错则更新失败
                return Result.error(code, "调度中心出错: " + content);
            }
            return Result.OK("执行一次成功");
        } catch (Exception e) {
            log.error(e.getMessage(), e);
            return Result.error("执行一次失败: " + e.getMessage());
        }
    }

    /**
     * 根据类型启动定时任务
     *
     * @param jobId  定时任务id
     * @param dataId 数据id
     * @param type   类型: 1 数据采集 2 数据交换 3 数据分析
     * @return Result
     */
    @Override
    public Result<Object> resumeByDataId(String jobId, String dataId, String type) {
        try {
            switch (type) {
                case BusinessConstant.TYPE_COLLECT:
                    DataCollect dataCollect = dataCollectService.getById(dataId);
                    dataCollect.setValid(BusinessConstant.VALID_DEFAULT);
                    dataCollect.setExecuteStatus(BusinessConstant.STATUS_2);
                    dataCollectService.updateById(dataCollect);
                    break;
                case BusinessConstant.TYPE_EXCHANGE:
                    DataExchange dataExchange = dataExchangeService.getById(dataId);
                    dataExchange.setValid(BusinessConstant.VALID_DEFAULT);
                    dataExchange.setExecuteStatus(BusinessConstant.STATUS_2);
                    dataExchangeService.updateById(dataExchange);
                    break;
                case BusinessConstant.TYPE_ANALYSE:
                    DataAnalyse dataAnalyse = dataAnalyseService.getById(dataId);
                    dataAnalyse.setExecuteStatus(BusinessConstant.STATUS_2);
                    dataAnalyse.setValid(BusinessConstant.VALID_DEFAULT);
                    dataAnalyseService.updateById(dataAnalyse);
                    break;
                default:
                    return Result.error("未匹配到类型");
            }
            JSONObject jsonObject = DispatchUtils.execute(jobId, dataId, BusinessConstant.JOB_START);
            Integer code = jsonObject.getInteger(BusinessConstant.RESULT_CODE);
            String content = jsonObject.getString(BusinessConstant.RESULT_CONTENT);
            if (CommonConstant.SC_INTERNAL_SERVER_ERROR_500.equals(code)) {
                // 调度中心出错则更新失败
                return Result.error(code, "调度中心出错: " + content);
            }
            return Result.OK("启动成功");
        } catch (Exception e) {
            log.error(e.getMessage(), e);
            return Result.error("启动失败: " + e.getMessage());
        }
    }

    /**
     * 根据类型停止定时任务
     *
     * @param jobId  定时任务id
     * @param dataId 数据id
     * @param type   类型: 1 数据采集 2 数据交换 3 数据分析
     * @param flag   标志: 不为空则立即停止任务
     * @return Result
     */
    @Override
    public Result<Object> pauseByDataId(String jobId, String dataId, String type, String flag) {
        try {
            switch (type) {
                case BusinessConstant.TYPE_COLLECT:
                    DataCollect dataCollect = dataCollectService.getById(dataId);
                    dataCollect.setValid(BusinessConstant.VALID_DISABLE);
                    dataCollect.setExecuteStatus(BusinessConstant.STATUS_1);
                    dataCollectService.updateById(dataCollect);
                    break;
                case BusinessConstant.TYPE_EXCHANGE:
                    DataExchange dataExchange = dataExchangeService.getById(dataId);
                    dataExchange.setValid(BusinessConstant.VALID_DISABLE);
                    dataExchange.setExecuteStatus(BusinessConstant.STATUS_1);
                    dataExchangeService.updateById(dataExchange);
                    break;
                case BusinessConstant.TYPE_ANALYSE:
                    DataAnalyse dataAnalyse = dataAnalyseService.getById(dataId);
                    dataAnalyse.setValid(BusinessConstant.VALID_DISABLE);
                    dataAnalyse.setExecuteStatus(BusinessConstant.STATUS_1);
                    dataAnalyseService.updateById(dataAnalyse);
                    break;
                default:
                    return Result.error("未匹配到类型");
            }

            JSONObject jsonObject = DispatchUtils.execute(jobId, dataId, BusinessConstant.JOB_STOP);
            if (flag != null) {
                jsonObject = DispatchUtils.execute(jobId, dataId, BusinessConstant.JOB_Immed_Stop);
            }
            Integer code = jsonObject.getInteger(BusinessConstant.RESULT_CODE);
            String content = jsonObject.getString(BusinessConstant.RESULT_CONTENT);
            if (CommonConstant.SC_INTERNAL_SERVER_ERROR_500.equals(code)) {
                // 调度中心出错则更新失败
                return Result.error(code, "调度中心出错: " + content);
            }
            return Result.OK("停止成功");
        } catch (Exception e) {
            log.error(e.getMessage(), e);
            return Result.error("停止失败: " + e.getMessage());
        }
    }

    @Override
    public Result<?> getCurrentLogInfo(String jobId) {
        JSONObject jsonObject = DispatchUtils.getCurrentLogInfo(jobId);
        Integer code = jsonObject.getInteger(BusinessConstant.RESULT_CODE);
        String content = jsonObject.getString(BusinessConstant.RESULT_CONTENT);
        if (CommonConstant.SC_INTERNAL_SERVER_ERROR_500.equals(code)) {
            // 调度中心出错则更新失败
            return Result.error(code, "调度中心出错: " + content);
        }
        return Result.OK(JSONObject.parse(content));
    }

    /**
     * 实时日志详情
     *
     * @param executorAddress
     * @param triggerTime
     * @param logId
     * @param fromLineNum
     * @return
     */
    @Override
    public Result<?> logDetailCat(String executorAddress, long triggerTime, long logId, int fromLineNum) {
        JSONObject jsonObject = DispatchUtils.logDetailCat(executorAddress, triggerTime, logId, fromLineNum);
        Integer code = jsonObject.getInteger(BusinessConstant.RESULT_CODE);
        String content = jsonObject.getString(BusinessConstant.RESULT_CONTENT);
        if (CommonConstant.SC_INTERNAL_SERVER_ERROR_500.equals(code)) {
            // 调度中心出错则更新失败
            return Result.error(code, "调度中心出错: " + content);
        }
        return Result.OK(jsonObject);
    }

    @Override
    public Result<?> batchSync(String ids, String type) {
        if (StringUtils.isBlank(ids) || StringUtils.isBlank(type)) {
            return Result.error("ids 及 type 不能为空");
        }
        List<String> idList = Arrays.asList(ids.split(","));
        switch (type) {
            case BusinessConstant.TYPE_COLLECT:
                List<DataCollect> dataCollects = dataCollectService.listByIds(idList);
                for (DataCollect dataCollect : dataCollects) {
                    // 生成定时任务
                    JSONObject jsonObject = DispatchUtils.addOrEditTask(null, dataCollect.getTaskName(),
                            collectHandler, dataCollect.getCornString(), dataCollect.getId(), dataCollect.getId());
                    String content = jsonObject.getString(BusinessConstant.RESULT_CONTENT);
                    dataCollect.setQuartzJobId(content);
                    this.dataCollectService.updateById(dataCollect);
                    log.info("同步成功，任务名称：" + dataCollect.getTaskName() + "，返回内容：" + content);
                }
                break;
            case BusinessConstant.TYPE_EXCHANGE:
                List<DataExchange> dataExchanges = dataExchangeService.listByIds(idList);
                for (DataExchange dataExchange : dataExchanges) {
                    JSONObject jsonObject = DispatchUtils.addOrEditTask(null, dataExchange.getTaskName(),
                            exchangeHandler, dataExchange.getCornString(), dataExchange.getId(), dataExchange.getId());
                    String content = jsonObject.getString(BusinessConstant.RESULT_CONTENT);
                    dataExchange.setQuartzJobId(content);
                    this.dataExchangeService.updateById(dataExchange);
                    log.info("同步成功，任务名称：" + dataExchange.getTaskName() + "，返回内容：" + content);
                }
                break;
            case BusinessConstant.TYPE_ANALYSE:
                List<DataAnalyse> dataAnalyses = dataAnalyseService.listByIds(idList);
                for (DataAnalyse dataAnalyse : dataAnalyses) {
                    JSONObject jsonObject = DispatchUtils.addOrEditTask(null, dataAnalyse.getTaskName(),
                            analyseHandler, dataAnalyse.getCornString(), dataAnalyse.getId(), dataAnalyse.getId());
                    String content = jsonObject.getString(BusinessConstant.RESULT_CONTENT);
                    dataAnalyse.setQuartzJobId(content);
                    this.dataAnalyseService.updateById(dataAnalyse);
                    log.info("同步成功，任务名称：" + dataAnalyse.getTaskName() + "，返回内容：" + content);
                }
                break;
            default:
                return Result.error("未匹配到类型");
        }
        return Result.OK("更新成功");
    }
}

```

### 7.远程调取工具类
```javascript

import cn.hutool.http.HttpUtil;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.google.common.collect.Maps;
import org.apache.commons.lang.StringUtils;
import org.apache.shiro.SecurityUtils;
import org.jeecg.common.constant.BusinessConstant;
import org.jeecg.common.system.vo.LoginUser;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

/**
 * 远程调用调度中心
 *
 * @author Ma·JJ
 * @date 2021-07-27
 */
@Component
public class DispatchUtils {

    private static String jobAddresses;
    private static String jobGroup;

    /**
     * 调度中心地址
     *
     * @param addresses
     */
    @Value("${xxl.job.admin.addresses}")
    public void setJobAddresses(String addresses) {
        jobAddresses = addresses;
    }

    /**
     * 默认执行器id
     *
     * @param jobGroupId
     */
    @Value("${xxl.job.jobGroupId}")
    public void setJobGroup(String jobGroupId) {
        jobGroup = jobGroupId;
    }


    /**
     * 远程添加/编辑任务(默认BEAN模式)
     *
     * @param jobId         xxlJobInfo主键
     * @param jobName       任务名称
     * @param jobHandler    对应执行器中开发的JobHandler类“@JobHandler”注解自定义的value值
     * @param scheduleConf  cron表达式
     * @param executorParam 执行参数
     * @param dataId        采集/分析/交换 id
     * @return JSONObject   code:状态码 content 返回信息
     */
    public static JSONObject addOrEditTask(String jobId, String jobName, String jobHandler,
                                           String scheduleConf, String executorParam, String dataId) {
        LoginUser sysUser = (LoginUser) SecurityUtils.getSubject().getPrincipal();
        HashMap<String, Object> map = Maps.newHashMap();
        String url = jobAddresses + "/jobinfo/add";
        map.put("addTime", new Date());
        if (StringUtils.isNotBlank(jobId)) {
            url = jobAddresses + "/jobinfo/update";
            map.put("id", Integer.valueOf(jobId));
            map.put("updateTime", new Date());
            map.put("addTime", null);
        }
        map.put("jobGroup", jobGroup);
        map.put("jobDesc", jobName);
        map.put("author", sysUser.getRealname());
        map.put("alarmEmail", sysUser.getEmail());
        map.put("scheduleType", "CRON");
        map.put("scheduleConf", scheduleConf);
        map.put("misfireStrategy", "DO_NOTHING");
        map.put("executorRouteStrategy", "FIRST");
        map.put("executorHandler", jobHandler);
        map.put("executorParam", executorParam);
        map.put("executorBlockStrategy", "SERIAL_EXECUTION");
        map.put("executorFailRetryCount", 0);
        map.put("glueType", "BEAN");
        map.put("glueSource", "");
        map.put("glueRemark", "GLUE代码初始化");
        map.put("glueUpdatetime", new Date());
        map.put("childJobId", null);
        map.put("dataId", dataId);
        String out = HttpUtil.post(url, map);
        return JSON.parseObject(out);
    }

    /**
     * 启动、停止、执行一次、删除任务
     *
     * @param jobId  任务id
     * @param dataId 数据id
     * @return type 类型  常量>>>BusinessConstant.JOB_
     */
    public static JSONObject execute(String jobId, String dataId, String type) {
        String url = jobAddresses + "/jobinfo";
        String result;
        HashMap<String, Object> map = Maps.newHashMap();
        map.put("id", jobId);
        switch (type) {
            case BusinessConstant.JOB_TRIGGER:
                map.put("executorParam", dataId);
                result = HttpUtil.post(url + "/trigger", map);
                break;
            case BusinessConstant.JOB_START:
                result = HttpUtil.post(url + "/start", map);
                break;
            case BusinessConstant.JOB_STOP:
                result = HttpUtil.post(url + "/stop", map);
                break;
            case BusinessConstant.JOB_REMOVE:
                result = HttpUtil.post(url + "/remove", map);
                break;
            case BusinessConstant.JOB_REMOVE_BATCH:
                String[] items = jobId.split(",");
                List<Integer> ids = Stream.of(items).map(Integer::parseInt).collect(Collectors.toList());
                map.put("ids", ids);
                result = HttpUtil.post(url + "/removeBatch", map);
                break;
            case BusinessConstant.JOB_Immed_Stop:
                result = HttpUtil.post(jobAddresses + "/joblog/killLog", map);
                break;
            default:
                result = "";
        }
        return JSON.parseObject(result);
    }

    /**
     * 获取实时日志信息
     *
     * @param jobId 任务id
     * @return
     */
    public static JSONObject getCurrentLogInfo(String jobId) {
        String url = jobAddresses + "/joblog/getCurrentLogInfo";
        HashMap<String, Object> map = Maps.newHashMap();
        map.put("jobId", Integer.parseInt(jobId));
        String out = HttpUtil.post(url, map);
        return JSON.parseObject(out);
    }

    public static JSONObject logDetailCat(String executorAddress, long triggerTime, long logId, int fromLineNum) {
        String url = jobAddresses + "/joblog/logDetailCat";
        HashMap<String, Object> map = Maps.newHashMap();
        map.put("executorAddress", executorAddress);
        map.put("triggerTime", triggerTime);
        map.put("logId", logId);
        map.put("fromLineNum", fromLineNum);
        String out = HttpUtil.post(url, map);
        return JSON.parseObject(out);
    }

}

```
