<script>
	import { onMount, onDestroy } from 'svelte';
	import { guideState, guideActions } from '@sudoku/stores/guide';
	
	let targetPosition = null;
	let currentGuide = { active: false, step: 0 };
	
	// 订阅store变化
	onMount(() => {
		const unsubscribe = guideState.subscribe(state => {
			currentGuide = state;
			// 状态变化后，延迟查找元素
			setTimeout(findTargetElement, 100);
		});
		
		return () => unsubscribe();
	});
	
	// 查找目标元素
	function findTargetElement() {
		if (!currentGuide.active) {
			targetPosition = null;
			return;
		}
		
		let element = null;
		if (currentGuide.step === 1) {
			// 第一步：找菜单按钮
			element = document.querySelector('.dropdown-button');
			console.log('第一步：查找菜单按钮', element ? '找到' : '未找到');
		} else if (currentGuide.step === 2) {
			// 第二步：找Enter Code选项
			element = document.querySelector('a[href="/enter-code"]');
			console.log('第二步：查找Enter Code', element ? '找到' : '未找到');
		}
		
		if (element) {
			const rect = element.getBoundingClientRect();
			targetPosition = {
				x: rect.left + rect.width / 2,
				y: rect.top + rect.height / 2,
				width: rect.width,
				height: rect.height
			};
			console.log('元素位置：', targetPosition);
		} else {
			targetPosition = null;
			console.log('未找到目标元素');
		}
	}
	
	// 监听窗口变化
	function handleResize() {
		if (currentGuide.active) {
			findTargetElement();
		}
	}
	
	onMount(() => {
		window.addEventListener('resize', handleResize);
		window.addEventListener('scroll', handleResize);
	});
	
	onDestroy(() => {
		window.removeEventListener('resize', handleResize);
		window.removeEventListener('scroll', handleResize);
	});
</script>

{#if currentGuide.active && targetPosition}
	<div class="guide-overlay">
		<!-- 遮罩背景 -->
		<div class="overlay-bg"></div>
		
		<!-- 高亮框 -->
		<div 
			class="highlight-box"
			style="
				left: {targetPosition.x - targetPosition.width/2 - 8}px;
				top: {targetPosition.y - targetPosition.height/2 - 8}px;
				width: {targetPosition.width + 16}px;
				height: {targetPosition.height + 16}px;
			"
		></div>
		
		{#if currentGuide.step === 1}
			<!-- 第一步：指引菜单按钮 -->
			<div class="arrow" style="left: {targetPosition.x}px; top: {targetPosition.y + targetPosition.height/2 + 25}px;">
				<svg width="40" height="30"><path d="M20,0 L20,20 M20,20 L10,10 M20,20 L30,10" stroke="#FF9800" stroke-width="3" fill="none"/></svg>
			</div>
			
			<div class="tooltip" style="left: {targetPosition.x - 100}px; top: {targetPosition.y + targetPosition.height/2 + 60}px;">
				<h3>📂 打开菜单</h3>
				<p>点击这里展开功能菜单</p>
				<button on:click={guideActions.next}>下一步 →</button>
			</div>
		
		{:else if currentGuide.step === 2}
			<!-- 第二步：指引Enter Code -->
			<div class="arrow" style="left: {targetPosition.x + targetPosition.width/2 + 20}px; top: {targetPosition.y}px;">
				<svg width="50" height="30"><path d="M0,15 L40,15 M40,15 L30,5 M40,15 L30,25" stroke="#4CAF50" stroke-width="3" fill="none"/></svg>
			</div>
			
			<div class="tooltip" style="left: {targetPosition.x + targetPosition.width/2 + 80}px; top: {targetPosition.y - 50}px;">
				<h3>📥 导入题目</h3>
				<p>点击 <strong>Enter Code</strong></p>
				<p>可以导入数独编码或Wiki上的数独URL</p>
				<button on:click={guideActions.complete}>点击此处开始使用</button>
			</div>
		{/if}
	</div>
{/if}



<style>
	.guide-overlay {
		position: fixed;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
		z-index: 10000;
		pointer-events: auto;
	}
	
	.overlay-bg {
		position: absolute;
		width: 100%;
		height: 100%;
		background: rgba(0, 0, 0, 0.6);
	}
	
	/* 高亮框 */
	.highlight-box {
		position: absolute;
		border: 2px solid #FF9800;
		border-radius: 6px;
		background: rgba(255, 152, 0, 0.1);
		box-shadow: 0 0 0 9999px rgba(0, 0, 0, 0.6),
					0 0 15px rgba(255, 152, 0, 0.6);
		pointer-events: none;
		animation: pulse 1.5s infinite;
	}
	
	/* 箭头 */
	.arrow {
		position: absolute;
		pointer-events: none;
		transform: translateX(-50%);
	}
	
	/* 提示框 */
	.tooltip {
		position: absolute;
		background: white;
		padding: 18px;
		border-radius: 10px;
		min-width: 220px;
		box-shadow: 0 6px 25px rgba(0, 0, 0, 0.25);
	}
	
	.tooltip h3 {
		margin: 0 0 10px 0;
		color: #2c3e50;
	}
	
	.close-btn {
		background: #4CAF50;
		color: white;
		border: none;
		padding: 10px 20px;
		border-radius: 6px;
		cursor: pointer;
		font-weight: bold;
		margin-top: 12px;
		width: 100%;
		transition: background 0.2s;
	}
	
	.close-btn:hover {
		background: #45a049;
	}
	
	@keyframes pulse {
		0% { box-shadow: 0 0 0 0 rgba(255, 152, 0, 0.7), 
						0 0 0 9999px rgba(0, 0, 0, 0.6); }
		70% { box-shadow: 0 0 0 10px rgba(255, 152, 0, 0), 
						0 0 0 9999px rgba(0, 0, 0, 0.6); }
		100% { box-shadow: 0 0 0 0 rgba(255, 152, 0, 0), 
						0 0 0 9999px rgba(0, 0, 0, 0.6); }
	}
</style>