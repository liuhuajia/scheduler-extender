# scheduler-extender

### predicates.go
只是对示例代码中predicates.go中的pod被接受的逻辑进行了修改。  
新逻辑是，获取pod申请时的时间代替随机数，当时间为偶数时，则节点时lucky的，可以被批准，否则拒绝批准该节点。

	func LuckyPredicate(pod *v1.Pod, node v1.Node) (bool, []string, error) {
		time_now=time.Now().UTC().UnixNano()
		lucky := (time_now%2) == 0
		if lucky {
			log.Printf("pod %v/%v is lucky to fit on node %v\n", pod.Name, pod.Namespace, node.Name)
			return true, nil, nil
		}
		log.Printf("pod %v/%v is unlucky to fit on node %v\n", pod.Name, pod.Namespace, node.Name)
		return false, []string{LuckyPredFailMsg}, nil
	}
### prioritize.go
由于在这一次的实验中只有一个master节点，故没有触发调度器的打分.
prioritize函数打分的逻辑和上面有些类似，若当前时间为偶数，则获得分值为1的幸运分，否则幸运分为0；再将获得的幸运分与随机分数相加获得节点的分数。

	func prioritize(args schedulerapi.ExtenderArgs) *schedulerapi.HostPriorityList {
		pod := args.Pod
		nodes := args.Nodes.Items

		hostPriorityList := make(schedulerapi.HostPriorityList, len(nodes))
		for i, node := range nodes {
			time_now=time.Now().UTC().UnixNano()
			score_lucky := (time_now%2) == 0
			if score_lucky {score := rand.Intn(schedulerapi.MaxPriority + 1)+1}
			else{score := rand.Intn(schedulerapi.MaxPriority + 1)}
			log.Printf(luckyPrioMsg, pod.Name, pod.Namespace, score)
			hostPriorityList[i] = schedulerapi.HostPriority{
				Host:  node.Name,
				Score: score,
			}
		}

		return &hostPriorityList
	}
