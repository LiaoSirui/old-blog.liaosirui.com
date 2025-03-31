## anacron

`anacron` （如 `cron` ）是一项允许您定期调度运行任务（通常称为作业）的服务。但是 `，anacron` 与 `cron` 有两种不同之处：

- 如果系统未在计划的时间运行，`anacron` 作业会延迟到系统运行；
- `anacron` 作业最多可以每天运行一次。

## 参考文档

- <https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/s1-scheduling_a_recurring_asynchronous_job_using_anacron>