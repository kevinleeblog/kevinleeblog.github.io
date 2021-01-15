---
layout: post
title: "llitek ILI2511 kernel crash issue"
auther: Kevin Lee
category: project1
tags: [ Android Open Source Project, MSM8953, Qualcomm ]
subtitle: 
visualworkflow: true
---

最近在忙把公司的Android 9升級到Android 10，方式就是先從代理商那邊拿到Android 10 for msm8953的codebase然後編譯看看有沒有問題， 然後再把LCM和Touchscreen的功能加上去

本想說一切應該會很順利，因為硬體在Android 9時都已經驗證過了，想不到在Android 10開機初始化Touchscreen竟發生了kernel panic導致watchdog啟動而reboot

Touchscreen driver使用的就是llitek ILI2511 driver，同一隻driver之前在Android 7就曾發生過莫名原因的當機，說是莫名是因為沒有任何訊息秀出來，只是感覺機器當掉約5秒就自動重啟了，後來同事使用delay大法當作workaround而結案，然而在Android 9上面一直都是正常的，直到Android10...

但相比於Android 7的莫名好了，Android 10是有秀出kernal panic的，這樣可以幫助Debug

```
[    4.728936] do not call blocking ops when !TASK_RUNNING; state=1 set at [<000000009481d7c4>] ilitek_irq_handle_thread+0x98/0x1344
[    4.728945] ------------[ cut here ]------------
[    4.728955] WARNING: CPU: 6 PID: 325 at /home/kevinlee/Desktop/MH5108_SC600/kernel/msm-4.9/kernel/sched/core.c:8436 __might_sleep+0x84/0x8c
[    4.728957] CHRDEV "ilitek_file" major number 228 goes below the dynamic allocation range
[    4.728963] Modules linked in:
[    4.728964]  ILITEK INFO line = 2556 ilitek_create_tool_node : register chrdev(228, 0)
[    4.728971] CPU: 6 PID: 325 Comm: ilitek_irq_thre Not tainted 4.9.186 #4
[    4.728974] Hardware name: Qualcomm Technologies, Inc. SDM450 NOPMI MTP (DT)
[    4.728979] task: 000000004569382d task.stack: 000000001d501a33
[    4.728984] PC is at __might_sleep+0x84/0x8c
[    4.728988] LR is at __might_sleep+0x84/0x8c
[    4.728992] pc : [<ffffff8884af2d6c>] lr : [<ffffff8884af2d6c>] pstate: 60400145
[    4.728996] sp : ffffffe9a7b03c90
[    4.729006] x29: ffffffe9a7b03c90 x28: 0000000000000000 
[    4.729016] x27: 0000000000000000 x26: ffffff888696e300 
[    4.729025] x25: 0000000000000000 x24: ffffff88873b7000 
[    4.729035] x23: ffffff888696e2f8 x22: 0000000000000000 
[    4.729044] x21: 0000000000000000 x20: 0000000000000636 
[    4.729053] x19: ffffff8886007b68 x18: ffffff8885bdf000 
[    4.729063] x17: 0000000000000006 x16: ffffffe9b170a4d8 
[    4.729072] x15: ffffffe9a7aa1080 x14: 3439303030303030 
[    4.729082] x13: 30303c5b20746120 x12: 74657320313d6574 
[    4.729091] x11: 617473203b474e49 x10: 4e4e55525f4b5341 
[    4.729101] x9 : 0000000000000006 x8 : ffffff8886374090 
[    4.729110] x7 : ffffffe9b1702090 x6 : 0000000000000000 
[    4.729120] x5 : ffffffe9a7b03ae0 x4 : ffffff8884b32708 
[    4.729129] x3 : 0000000000000000 x2 : 0000000000040900 
[    4.729138] x1 : 0000000000040900 x0 : 0000000000000075 
[    4.729142] 
[    4.729142] PC: 0xffffff8884af2d2c:
[    4.729175] 2d2c  34000121 aa1303e0 2a1403e1 2a1503e2 97ffff9f f94013f5 a94153f3 a8c37bfd
[    4.729206] 2d4c  d65f03c0 f9402c01 d0009f20 52800025 aa0203e3 910d6000 39001085 94040a1c
[    4.729238] 2d6c  d4210000 17fffff0 a9be7bfd 910003fd a90153f3 aa0003f3 aa1e03e0 d503201f
[    4.729269] 2d8c  b9496260 940030a2 f9447261 eb010002 540002e4 f9047260 b000eb20 91260000
[    4.729273] 
[    4.729273] LR: 0xffffff8884af2d2c:
[    4.729304] 2d2c  34000121 aa1303e0 2a1403e1 2a1503e2 97ffff9f f94013f5 a94153f3 a8c37bfd
[    4.729336] 2d4c  d65f03c0 f9402c01 d0009f20 52800025 aa0203e3 910d6000 39001085 94040a1c
[    4.729367] 2d6c  d4210000 17fffff0 a9be7bfd 910003fd a90153f3 aa0003f3 aa1e03e0 d503201f
[    4.729399] 2d8c  b9496260 940030a2 f9447261 eb010002 540002e4 f9047260 b000eb20 91260000
[    4.729403] 
[    4.729403] SP: 0xffffffe9a7b03c50:
[    4.729434] 3c50  84af2d6c ffffff88 a7b03c90 ffffffe9 84af2d6c ffffff88 60400145 00000000
[    4.729466] 3c70  869f8bc1 ffffff88 00000001 00000000 ffffffff ffffffff b1702090 ffffffe9
[    4.729497] 3c90  a7b03cd0 ffffffe9 85415430 ffffff88 873b7000 ffffff88 a7aa0f80 ffffffe9
[    4.729529] 3cb0  85ed3978 ffffff88 854153e4 ffffff88 aadc3e00 ffffffe9 85b4335c ffffff88
[    4.729533]  ILITEK INFO line = 431 ilitek_touch_driver_init : add touch device driver i2c driver.
[    4.729538] ---[ end trace 50de212d48db1a02 ]---
[    4.729543] Call trace:
[    4.729548] Exception stack(0xffffffe9a7b03a90 to 0xffffffe9a7b03bc0)
[    4.729553] 3a80:                                   ffffff8886007b68 0000007fffffffff
[    4.729559] 3aa0: 0000000012a14000 ffffff8884af2d6c 0000000060400145 000000000000003d
[    4.729565] 3ac0: 0000000000000000 ffffff888696e300 00000000ffffffff ffffff8886881ac8
[    4.729571] 3ae0: ffffffe9a7b03b00 ffffff8884b3b3c4 ffffff8886881ac8 ffffff8886881a38
[    4.729576] 3b00: ffffffe9a7b03b20 ffffff8884b3d6e4 0000000000000075 ffffff8886881a38
[    4.729582] 3b20: ffffffe9a7b03bc0 ffffff8884bf5680 ffffff8886856000 ffffff8885ed8358
[    4.729588] 3b40: 0000000000000000 0000000000000000 ffffff888696e2f8 0000000000040900
[    4.729593] 3b60: 0000000000000075 0000000000040900 0000000000040900 0000000000000000
[    4.729599] 3b80: ffffff8884b32708 ffffffe9a7b03ae0 0000000000000000 ffffffe9b1702090
[    4.729605] 3ba0: ffffff8886374090 0000000000000006 4e4e55525f4b5341 617473203b474e49
[    4.729611] [<000000007b6bc695>] __might_sleep+0x84/0x8c
[    4.729617] [<000000000a1549e5>] ilitek_irq_handle_thread+0xcc/0x1344
[    4.729623] [<000000001e7ef2c5>] kthread+0xf4/0x108
[    4.729630] [<00000000df491dbe>] ret_from_fork+0x10/0x40
```

由Call trace的函式來看問題發生在ilitek_irq_handle_thread函式內

```c
static int ilitek_irq_handle_thread(void *arg) {
	int ret=0;
	struct sched_param param = { .sched_priority = 4};
	sched_setscheduler(current, SCHED_RR, &param);	
	tp_log_info("%s, enter\n", __func__);

	// mainloop
	while(!kthread_should_stop() && !ilitek_exit_report){
		set_current_state(TASK_INTERRUPTIBLE);
		wait_event_interruptible(waiter, ilitek_data->irq_trigger);
		ilitek_data->irq_trigger = false;
		set_current_state(TASK_RUNNING);
		if (ilitek_i2c_process_and_report() < 0){
			tp_log_err("process error\n");
		}
		ilitek_irq_enable();
	}
	tp_log_err("%s, exit\n", __func__);
	tp_log_err("%s, exit\n", __func__);
	tp_log_err("%s, exit\n", __func__);
	return ret;
}
```

wait_event_interruptible和wake_up_interruptible是成對的函式
從函式原意來看，ilitek_irq_handle_thread是一個由kthread產生的另外一個thread
這個thread平常就是在等待touchscreen的irq pin被拉low，當irq pin拉low時，ilitek_data->irq_trigger = 1 
wait_event_interruptible在等待ilitek_data->irq_trigger = 1，條件滿足時，會繼續下一行並從ilitek_i2c_process_and_report取得座標

但壞就壞在wait_event_interruptible除了可被ilitek_data->irq_trigger = 1喚醒外，還多了signal也可喚醒
導致程式邏輯出錯而觸發Kernal panic
修正方式就是把wait_event_interruptible和wake_up_interruptible個別替換為wait_event()和wake_up()，會替換的原因就是確保被喚醒的條件只能依照ilitek_data->irq_trigger的值是否滿足，而不把signal也攪和進來。

還有下一段和ili2511相關的kernel panic，在往下看看是什麼原因造成的

```
[   18.861790] CPU: 3 PID: 657 Comm: HwBinder:583_1 Tainted: G        W  O    4.9.186 #4
[   18.861791] Hardware name: Qualcomm Technologies, Inc. SDM450 NOPMI MTP (DT)
[   18.861795] task: 000000007bf6d79f task.stack: 000000002ca93e24
[   18.861808] PC is at ___might_sleep+0x11c/0x130
[   18.861811] LR is at ___might_sleep+0x11c/0x130
[   18.861813] pc : [<ffffff8884af2cd4>] lr : [<ffffff8884af2cd4>] pstate: 604001c5
[   18.861814] sp : ffffffe99d833960
[   18.861818] x29: ffffffe99d833960 x28: ffffffe99fc46c80 
[   18.861821] x27: ffffffe99d833c30 x26: ffffffe99d833c38 
[   18.861824] x25: ffffffe99d833c70 x24: ffffff88868c89c0 
[   18.861827] x23: ffffff8886007000 x22: ffffff8885c21000 
[   18.861830] x21: 0000000000000000 x20: ffffff8885edc8f0 
[   18.861833] x19: ffffffe99fc46c80 x18: 00000079d70f8000 
[   18.861836] x17: 0000000000000003 x16: ffffffe9b16c54d8 
[   18.861839] x15: 0000000000000000 x14: 442f65656c6e6976 
[   18.861843] x13: 656b2f656d6f682f x12: ffffffe99d833960 
[   18.861846] x11: ffffffe99d833960 x10: ffffffe99d833960 
[   18.861849] x9 : ffffff8886a8a178 x8 : ffffffe9b1592784 
[   18.861852] x7 : 0000000000000000 x6 : 0000000010513706 
[   18.861855] x5 : 0000000000000015 x4 : 00000000012e06ed 
[   18.861858] x3 : 0000000000000004 x2 : 0000000000040900 
[   18.861861] x1 : 0000000057ac6e9d x0 : ffffffe99fc46c80 
[   18.861863] 
[   18.861863] PC: 0xffffff8884af2c94:
[   18.861869] 2c94  d0009f20 1a9f07e1
[   18.861869] msm_audio_get_copp_idx_from_port_id: Invalid FE, exiting
[   18.861875]  910ba000 12190042 91216264[   18.861875] msm_audio_sound_focus_get: Could not get copp idx for port_id=16385
[   18.861879]  94040a4c f9403261 d28dd3a0
2cb4  f2aaf580 f9400021 eb00003f 540000c1 d53b4220 36380060 d5384100 94032994
[   18.861900] 2cd4  d4210000 d0009f20 910ca000 94040a3e 17fffff8 a9bd7bfd 910003fd a90153f3
[   18.861910] 2cf4  f90013f5 aa0003f3 aa1e03e0 2a0103f4 2a0203f5 d503201f d5384100 f9402c01
[   18.861912] 
[   18.861912] LR: 0xffffff8884af2c94:
[   18.861922] 2c94  d0009f20 1a9f07e1 910ba000 12190042 91216264 94040a4c f9403261 d28dd3a0
[   18.861932] 2cb4  f2aaf580 f9400021 eb00003f 540000c1 d53b4220 36380060 d5384100 94032994
[   18.861942] 2cd4  d4210000 d0009f20 910ca000 94040a3e 17fffff8 a9bd7bfd 910003fd a90153f3
[   18.861952] 2cf4  f90013f5 aa0003f3 aa1e03e0 2a0103f4 2a0203f5 d503201f d5384100 f9402c01
[   18.861954] 
[   18.861954] SP: 0xffffffe99d833920:
[   18.861965] 3920  84af2cd4 ffffff88 9d833960 ffffffe9 84af2cd4 ffffff88 604001c5 00000000
[   18.861974] 3940  9fc474d8 ffffffe9 00000015 00000000 ffffffff 0000007f 00000000 00000000
[   18.861985] 3960  9d833990 ffffffe9 84af2d40 ffffff88 85edc8f0 ffffff88 0000006e 00000000
[   18.861995] 3980  a7dcb800 ffffffe9 00000000 00000000 9d8339c0 ffffffe9 84b3ff54 ffffff88
[   18.861997] Process HwBinder:583_1 (pid: 657, stack limit = 0x000000002ca93e24)
[   18.862000] Call trace:
[   18.862003] Exception stack(0xffffffe99d833760 to 0xffffffe99d833890)
[   18.862008] 3760: ffffffe99fc46c80 0000007fffffffff 0000000012a14000 ffffff8884af2cd4
[   18.862011] 3780: 00000000604001c5 000000000000003d ffffffe99d833c70 ffffffe99d833c38
[   18.862015] 37a0: 00000000ffffffff ffffff8886881ac8 ffffffe99d8337d0 ffffff8884b3b3c4
[   18.862018] 37c0: ffffff8886881ac8 ffffff8886881a38 ffffffe99d8337f0 ffffff8884b3d6e4
[   18.862022] 37e0: 0000000000000044 ffffff8886881a38 ffffffe99d833890 ffffff8884bf5680
[   18.862025] 3800: ffffff8886856000 ffffff8885ed82e8 0000000000000000 ffffff8885c21000
[   18.862029] 3820: ffffff8886007000 0000000000040900 ffffffe99fc46c80 0000000057ac6e9d
[   18.862032] 3840: 0000000000040900 0000000000000004 00000000012e06ed 0000000000000015
[   18.862036] 3860: 0000000010513706 0000000000000000 ffffffe9b1592784 ffffff8886a8a178
[   18.862038] 3880: ffffffe99d833960 ffffffe99d833960
[   18.862042] [<000000006cc83ebe>] ___might_sleep+0x11c/0x130
[   18.862045] [<0000000010ebf6b7>] __might_sleep+0x58/0x8c
[   18.862051] [<00000000d5ee355d>] synchronize_irq+0x50/0xbc
[   18.862055] [<0000000038840f53>] disable_irq+0x6c/0x98
[   18.862062] [<000000009c2d2884>] ilitek_irq_disable+0x58/0x9c
[   18.862065] [<00000000e984d1ad>] ilitek_resume+0xec/0x1ec
[   18.862068] [<0000000030bc9cbf>] ilitek_fb_notifier_callback+0x68/0xa0
[   18.862073] [<00000000e5c627cb>] blocking_notifier_call_chain+0x6c/0xbc
[   18.862078] [<0000000089a9c23a>] fb_notifier_call_chain+0x30/0x3c
[   18.862081] [<000000009cbc7251>] fb_blank+0xc0/0xd4
[   18.862084] [<0000000066fbc64b>] do_fb_ioctl+0x2ac/0x6f4
[   18.862087] [<00000000e64811dc>] fb_ioctl+0x54/0x64
[   18.862092] [<00000000f0e75e62>] do_vfs_ioctl+0xd0/0xdb4
[   18.862095] [<00000000ead31333>] SyS_ioctl+0x90/0xa4
[   18.862099] [<00000000cb0a2a73>] el0_svc_naked+0x34/0x38
[   18.862104] Code: d53b4220 36380060 d5384100 94032994 (d4210000) 
[   18.862110] ---[ end trace 50de212d48db1a04 ]---
```

由Call trace來看順序是從ilitek_fb_notifier_callback -> ilitek_resume -> ilitek_irq_disable -> disable_irq

```c
void ilitek_irq_disable(void) {
    unsigned long irqflag = 0;
	spin_lock_irqsave(&ilitek_data->irq_lock, irqflag);
	if ((ilitek_data->irq_status)) {
#ifdef NO_USE_MTK_ANDROID_SDK_6_UPWARD
		mt_eint_mask(CUST_EINT_TOUCH_PANEL_NUM);
#else
        disable_irq(ilitek_data->client->irq);
#endif
		ilitek_data->irq_status = false;
		tp_log_info("\n");
	}
	spin_unlock_irqrestore(&ilitek_data->irq_lock, irqflag);
}
```

從上述程式中看到執行disable_irq()引發Kernel panic

推測是disable_irq會等待中斷處理函式結束後才會關閉irq並返回，但不知如何一直被block動彈不得導致系統死掉。把disable_irq換成disable_irq_nosync就可以解決此問題阿