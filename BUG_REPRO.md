# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

任务租约已经过期，但调度器扫描后仍不回收任务，任务长期保持已领取状态。请修复过期租约回收流程。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/portcoord-backend-qa-11
- 仓库地址：https://github.com/zhanglei10281852-gif/portcoord-backend-qa-11.git
- parent SHA：b547d2b40f4681b76ed8a3dad10e6ee6419fb768

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/portcoord-backend-qa-11.git bug-repro
cd bug-repro
git checkout --detach b547d2b40f4681b76ed8a3dad10e6ee6419fb768
go test ./internal/pilottask -run "^TestPilotTask_PreemptExpiredClaim$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/pilottask -run "^TestPilotTask_PreemptExpiredClaim$" -count=1
--- FAIL: TestPilotTask_PreemptExpiredClaim (0.01s)
    pilottask_test.go:180: expected 1 preempted, got 0
FAIL
FAIL	portcoord/internal/pilottask	0.010s
FAIL

```

stderr：

```text
warning: internal/pilottask/pilottask_test.go has type 100755, expected 100644
warning: internal/pilottask/pilottask_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/pilottask -run "^TestPilotTask_PreemptExpiredClaim$" -count=1
--- FAIL: TestPilotTask_PreemptExpiredClaim (0.25s)
    pilottask_test.go:180: expected 1 preempted, got 0
FAIL
FAIL	portcoord/internal/pilottask	0.464s
FAIL

```

stderr：

```text
warning: internal/pilottask/pilottask_test.go has type 100755, expected 100644
warning: internal/pilottask/pilottask_test.go has type 100755, expected 100644

```

## 通过条件

在题面触发条件下，公开行为必须恢复且原始异常不再出现；定向命令 go test ./internal/pilottask -run ^TestPilotTask_PreemptExpiredClaim$ -count=1、相关包测试、全量测试、race、vet 和 build 必须通过；不得删除或跳过测试，也不得绕过目标逻辑。
