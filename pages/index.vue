<template>
	<ElForm
		size="small"
		label-position="top"
		:model="config"
	>
		<!-- element-loading-custom-class="config-start"
			v-loading="mainStore.configStartEnable"
			:element-loading-spinner="'<path />'"
			element-loading-text="-------已经启用配置文件，界面配置不再生效-------" -->
		<!-- <div> -->
		<ElFormItem
			label="服务器"
			prop="serverUrl"
			class="full-label"
		>
			<template #label>
				<div class="flex items-center flex-nowrap gap-[0_5px]">
					<div>服务器</div>
					<span>-</span>
					<ElTag
						effect="dark"
						:type="data.isSuccessGetIp ? 'success' : 'info'"
					>
						{{ data.isSuccessGetIp ? "联机成功" : data.isStart && !data.isSuccessGetIp ? "联机中" : "未联机" }}
					</ElTag>
					<ElButton
						v-if="!data.coreVersion"
						@click="getCoreVersion(true)"
					>
						获取内核版本
					</ElButton>
					<div
						v-else
						class="flex-1 truncate"
					>
						<ElTooltip :content="data.coreVersion">
							<ElTag type="info">
								{{ data.coreVersion }}
							</ElTag>
						</ElTooltip>
					</div>

					<!-- <ElPopconfirm
						width="325"
						cancel-button-text="取消"
						confirm-button-text="继续"
						title="内核从github下载，需要出国工具，可能下载缓慢或失败，是否继续?
						也可以从官方群里手动下载后解压到easytier-game.exe同级目录下的easytier目录里,全部覆盖即可"
						@confirm="handleCoreManagement"
					>
						<template #reference> -->
					<ElButton
						class="ml-auto"
						:disabled="data.isStart"
						@click="handleCoreManagement"
						:loading="data.update"
						type="primary"
						size="small"
					>
						内核管理
						<!-- {{ data.coreVersion ? "更新内核" : "下载内核" }} -->
					</ElButton>
					<!-- </template>
					</ElPopconfirm> -->
				</div>
			</template>
			<ElSelect
				allow-create
				filterable
				placeholder="请选择服务器地址"
				default-first-option
				v-model="config.serverUrl"
				@change="handleServerUrlChange"
			>
				<template #prefix>
					<div :class="config.protocol && config.protocol.length > 1 ? 'w-[120px]' : 'w-[80px]'">
						<ElSelect
							placeholder="协议"
							multiple
							collapse-tags
							@click.stop
							v-model="config.protocol"
							@change="handleServerUrlChange"
						>
							<ElOption
								v-for="item in protocols"
								:key="item"
								:label="item"
								:value="item"
							></ElOption>
						</ElSelect>
					</div>
				</template>
				<ElOption
					v-for="item in mainStore.basePeers"
					:key="item"
					:label="item"
					:value="item"
				>
					<div class="flex items-center gap-[20px] overflow-hidden flex-nowrap max-w-[calc(100vw-62px)]">
						<ElTooltip :content="item">
							<p class="truncate">{{ item }}</p>
						</ElTooltip>
						<div class="flex-shrink-0 ml-auto">
							<ElButton
								@click.stop="handleDeleteServerUrl(item)"
								round
								:icon="Delete"
								type="danger"
							></ElButton>
						</div>
					</div>
				</ElOption>
			</ElSelect>
		</ElFormItem>
		<div class="flex flex-wrap gap-[0_10px] items-center">
			<div class="flex-1">
				<ElFormItem label="网络名">
					<template #label>
						<div class="flex items-center">
							网络名
							<ElTooltip content="对应命令行参数 --network-name">
								<ElIcon><QuestionFilled /></ElIcon>
							</ElTooltip>
						</div>
					</template>
					<ElInput
						maxlength="100"
						placeholder="请输入网络名"
						v-model="config.networkName"
					></ElInput>
				</ElFormItem>
			</div>
			<div class="flex-1">
				<ElFormItem label="网络密码">
					<template #label>
						<div class="flex items-center">
							网络密码
							<ElTooltip content="对应命令行参数 --network-secret">
								<ElIcon><QuestionFilled /></ElIcon>
							</ElTooltip>
						</div>
					</template>
					<ElInput
						show-password
						maxlength="100"
						placeholder="请输入网络密码"
						v-model="config.networkPassword"
						type="password"
					></ElInput>
				</ElFormItem>
			</div>
		</div>
		<div class="flex gap-[0_10px]">
			<ElFormItem label="主机名">
				<template #label>
					<div class="flex items-center">
						主机名
						<ElTooltip content="对应命令行参数 --hostname">
							<ElIcon><QuestionFilled /></ElIcon>
						</ElTooltip>
					</div>
				</template>
				<ElInput
					maxlength="100"
					placeholder="例如: Player1"
					v-model="config.hostname"
				></ElInput>
			</ElFormItem>
			<ElFormItem
				class="w-[70%]"
				label="局域网IP"
			>
				<template #label>
					<div class="flex items-center h-[20px]">
						虚拟网IP
						<ElTooltip content="对应命令行参数 --ipv4">
							<ElIcon><QuestionFilled /></ElIcon>
						</ElTooltip>
						<ElSwitch
							v-model="config.dhcp"
							class="ml-[5px]"
							inline-prompt
							inactive-text="固定IP"
							active-text="动态获取IP"
							size="small"
						></ElSwitch>
					</div>
				</template>
				<ElInput
					maxlength="100"
					:disabled="config.dhcp"
					:placeholder="data.isStart && config.dhcp ? '等待动态分配IP...' : '例如: 10.126.126.1'"
					v-model="config.ipv4"
				></ElInput>
			</ElFormItem>
		</div>
		<!-- </div> -->
		<div class="flex items-start">
			<div>
				<div>
					<ElDropdown
						@command="handleStartCommand"
						split-button
						trigger="click"
						size="default"
						:type="!data.isStart ? 'primary' : 'danger'"
						:disabled="data.startLoading || !data.coreVersion || data.update"
						@click="handleConnection"
					>
						{{ !data.isStart ? "启动联机" : "停止联机" }}
						<template #dropdown>
							<ElDropdownMenu>
								<ElDropdownItem
									command="import_config"
									:icon="Link"
								>
									导入配置
								</ElDropdownItem>
								<ElDropdownItem
									command="share_config"
									:icon="Share"
								>
									分享配置
								</ElDropdownItem>
								<ElDropdownItem
									:icon="Tools"
									command="toml"
									:disabled="data.isStart"
								>
									使用外部配置文件
								</ElDropdownItem>
							</ElDropdownMenu>
						</template>
					</ElDropdown>
				</div>
				<div class="mt-[6px] pl-[2px]">
					<ElTooltip
						placement="left"
						content="日志"
					>
						<ElButton
							:type="!data.logVisible ? 'info' : 'warning'"
							@click="handleShowLogDialog"
							:icon="List"
							size="small"
							plain
						></ElButton>
					</ElTooltip>
					<ElTooltip
						placement="left"
						content="成员"
					>
						<ElButton
							@click="handleShowMemberDialog"
							:icon="UserFilled"
							plain
							type="success"
							size="small"
						></ElButton>
					</ElTooltip>
				</div>
			</div>
			<div class="ml-auto">
				<ElCheckbox
					v-model="config.disbleP2p"
					size="small"
				>
					强制中转
				</ElCheckbox>
				<ElTooltip
					placement="top-start"
					content="自启后隐藏于托盘，不显示界面"
				>
					<ElCheckbox
						@change="handleAutoStartByTask"
						:model-value="config.autoStart"
						size="small"
					>
						开机自启
					</ElCheckbox>
				</ElTooltip>
				<div>
					<ElButton
						@click="handleShowCidrDialog"
						:icon="Share"
					>
						子网代理
					</ElButton>
					<ElButton
						@click="handleShowAdvanceDialog"
						:icon="Setting"
					>
						高级选项
					</ElButton>
				</div>
				<div class="flex items-center gap-[0_5px]">
					<div>
						<ElButton
							plain
							type="primary"
							size="small"
							:icon="MagicStick"
							@click="handleShowToolDialog"
						>
							增强工具
						</ElButton>
					</div>
					<ElLink
						class="!text-[9px] pb-[2px] ml-[8px] truncate"
						type="info"
						:underline="false"
						@click="open('https://github.com/EasyTier/EasytierGame')"
					>
						EasytierGame主页
					</ElLink>
				</div>
			</div>
		</div>
	</ElForm>
	<ElDialog
		width="95%"
		top="10px"
		append-to-body
		:z-index="10"
		class="!mb-0"
		v-model="coreManagementData.visible"
		:close-on-press-escape="false"
		title="内核管理"
	>
		<div>
			<ElText>当前版本: {{ data.coreVersion || "-" }}</ElText>
		</div>
		<div class="mt-[10px] pb-[5px]">
			<ElText class="!mr-[10px]">选择一个内核版本安装</ElText>
			<ElButton
				:loading="coreManagementData.loading"
				@click="getReleaseList"
				size="small"
			>
				刷新
			</ElButton>
		</div>
		<ElSelect
			placeholder="请选择内核版本"
			no-data-text="正在获取中..."
			popper-class="!h-[120px]"
			v-model="coreManagementData.data"
		>
			<ElOption
				v-for="(release, idx) in data.releaseList"
				:key="`${idx}-ray`"
				:label="release[0]"
				:value="`${release[2]}<>${release[1]}`"
			>
				{{ release ? release[0] : "" }}
			</ElOption>
		</ElSelect>
		<div class="pb-[5px] mt-[10px]">
			<ElTooltip content="不使用出国软件也能告诉下载github Release包的地址，可自行搜索替换使用">
				<ElText>github下载加速地址</ElText>
			</ElTooltip>
		</div>
		<ElInput
			v-model="mainStore.githubFastUrl"
			placeholder="请输入github加速地址"
		></ElInput>
		<template #footer>
			<div class="text-right">
				<ElButton
					size="small"
					:loading="data.update"
					@click="handleInstallCore"
					type="primary"
				>
					安装选中内核
				</ElButton>
				<ElButton
					size="small"
					type="warning"
					@click="addQQGroup"
				>
					加群获取
				</ElButton>
				<ElButton
					size="small"
					@click="coreManagementData.visible = false"
				>
					取消
				</ElButton>
			</div>
		</template>
	</ElDialog>
	<ElDialog
		width="95%"
		top="10px"
		append-to-body
		:z-index="10"
		class="!mb-0"
		v-model="importConfigData.visible"
		:close-on-press-escape="false"
		title="导入分享"
	>
		<ElInput
			type="textarea"
			:rows="6"
			placeholder="请粘贴分享的配置"
			v-model="importConfigData.data"
		></ElInput>
		<template #footer>
			<div>
				<el-text type="danger">导入成功后，您当前的部分配置将被替换</el-text>
			</div>
			<div class="text-right">
				<ElButton
					size="small"
					@click="handleStartImport"
					type="primary"
				>
					导入
				</ElButton>
			</div>
		</template>
	</ElDialog>
	<ElDialog
		width="95%"
		top="30px"
		append-to-body
		:z-index="10"
		class="!mb-0"
		v-model="configStart.visible"
		:close-on-press-escape="false"
		title="配置文件启动"
	>
		<div class="flex items-center gap-[0_4px]">
			<span>启用</span>
			<ElSwitch v-model="mainStore.configStartEnable"></ElSwitch>
			<ElTooltip content="启用后将完全使用选中的配置文件作为联机配置，其余界面配置不会生效">
				<ElIcon class="ml-[3px]"><QuestionFilled /></ElIcon>
			</ElTooltip>
			<ElButton
				@click="openConfigDir"
				size="small"
			>
				打开配置目录
			</ElButton>
			<ElButton
				@click="handleStartCommand('toml')"
				type="primary"
				:icon="RefreshRight"
				size="small"
			>
				刷新
			</ElButton>
		</div>
		<div class="mt-[5px]">
			<ElSelect
				v-model="mainStore.configPath"
				no-data-text="目录没有配置文件"
				placeholder="选择配置文件"
			>
				<ElOption
					v-for="item in configStart.list"
					:key="item.path"
					:value="item.path"
					:label="item.name"
				></ElOption>
			</ElSelect>
		</div>
		<template #footer>
			<div class="text-right">
				<ElButton
					size="small"
					@click="configStart.visible = false"
					type="danger"
				>
					关闭
				</ElButton>
			</div>
		</template>
	</ElDialog>
</template>
<script setup lang="ts">
	import { invoke } from "@tauri-apps/api/core";
	import { listen } from "@tauri-apps/api/event";
	import { open, Command } from "@tauri-apps/plugin-shell";
	import { QuestionFilled, Delete, List, UserFilled, Setting, Share, RefreshRight, Link, Tools, MagicStick } from "@element-plus/icons-vue";
	import { reactive, onBeforeUnmount, onMounted } from "vue";
	import { useTray, setTrayRunState, setTrayTooltip } from "~/composables/tray";
	import { initStartWinIpBroadcast } from "~/composables/netcard";
	import useMainStore from "@/stores/index";
	import { ElMessage, ElMessageBox } from "element-plus";
	import { getCurrentWindow } from "@tauri-apps/api/window";
	import { getAllWebviewWindows } from "@tauri-apps/api/webviewWindow";
	import etWindows from "@/composables/windows";
	import * as tauriAutoStart from "@tauri-apps/plugin-autostart";
	import { resourceDir as getResourceDir, join } from "@tauri-apps/api/path";
	import { readDir, exists, mkdir, BaseDirectory, readTextFile } from "@tauri-apps/plugin-fs";
	import { updateConfigJson } from "~/composables/configJson";
	import { writeText, readText } from "@tauri-apps/plugin-clipboard-manager";
	import { sortedUniq, uniq } from "lodash-es";
	import { bounce, addQQGroup } from "~/utils";

	let is_close = false;

	const tray = await useTray(true, async () => {
		is_close = true;
		await invoke("stop_command", { child_id: listenObj.thread_id || 0 });
		await invoke("stop_command", { child_id: data.winipBcPid || 0 });
	}, async () => {
		await handleConnection();
		return data.isStart
	});

	const mainStore = useMainStore();
	const config = mainStore.config;
	// console.error(config);
	const protocols = ["tcp", "udp", "ws", "wss", "wg", "quic"];
	const data = reactive<{ [key: string]: any; releaseList: Array<Array<string>> }>({
		logVisible: false,
		cidrVisible: false,
		advanceVisible: false,
		toolVisible: false,
		winipBcPid: 0, //WinIPBroadcast进程id
		winipBcStart: false,
		memberVisible: false,
		log: "",
		update: false,
		releaseList: [],
		coreVersion: "-",
		isSuccessGetIp: false,
		startLoading: false,
		isStart: false,
		connectionSuccess: false,
		configJsonSeverUrl: "" // 本地保存一次，用于回填config.json
	});

	const configStart = reactive<{ list: Array<{ path: string; name: string }>; [key: string]: any }>({
		visible: false,
		loading: false,
		list: [] //配置文件列表
	});

	const importConfigData = reactive<{ data: ""; [key: string]: any }>({
		visible: false,
		loading: false,
		data: ""
	});

	const coreManagementData = reactive<{ data: ""; [key: string]: any }>({
		visible: false,
		loading: false,
		data: ""
	});

	const closePrevent = async () => {
		const appWindow = getCurrentWindow();
		if (appWindow.label == "main") {
			appWindow.onCloseRequested(async event => {
				// console.error(appWindow.label);
				if (!is_close) {
					// console.error(1);
					event.preventDefault();
					appWindow.hide();
				}
			});
		}
	};

	const handleDeleteServerUrl = (url: string) => {
		const newBasePeers = [...mainStore.basePeers];
		const idx = newBasePeers.indexOf(url);
		if (idx >= 0) {
			if (config.serverUrl === url) {
				config.serverUrl = "";
			}
			newBasePeers.splice(idx, 1);
		}
		mainStore.basePeers = uniq([...newBasePeers]);
		if (mainStore.basePeers.length > 0) {
			mainStore.config.serverUrl = mainStore.basePeers[0];
		}
	};

	const handleServerUrlChange = () => {
		let inputProtocols: string | null = null;
		for (const p of protocols) {
			if (config.serverUrl.toLowerCase().startsWith(`${p}://`)) {
				inputProtocols = p;
				break;
			}
		}
		if (inputProtocols) {
			mainStore.config.protocol = [inputProtocols];
			mainStore.config.serverUrl = mainStore.config.serverUrl.slice(inputProtocols.length + 3);
		}
		mainStore.basePeers = uniq([mainStore.config.serverUrl, ...mainStore.basePeers]);
	};

	const listenObj: { [key: string]: any } = {
		unListenOutPut: null,
		unListenThreadId: null,
		unListenConfigStart: null,
		thread_id: null,
		async listenOutput() {
			// const appWindow = getCurrentWindow();
			const unListen = await listen("command-output", async event => {
				data.isStart = true;
				// console.error(event.payload);
				if (event.payload) {
					data.startLoading = false;
					let ipv4 = /dhcp ip changed. old: None, new: Some\((\d+\.\d+\.\d+\.\d+).*\)/g.exec(event.payload as string)?.[1];
					let devName = /tun device ready. dev: (.*)/g.exec(event.payload as string)?.[1];
					if (config.dhcp || mainStore.configStartEnable) {
						if (ipv4) {
							config.ipv4 = ipv4;
							await setTrayRunState(tray, true);
							data.isSuccessGetIp = true;
							await setTrayTooltip(tray, `IP: ${ipv4}`);
						}
					} else {
						if ((event.payload as string).includes("new peer connection added") && !data.isSuccessGetIp) {
							await setTrayRunState(tray, true);
							data.isSuccessGetIp = true;
							await setTrayTooltip(tray, `IP: ${config.ipv4}`);
						}
					}
					if (
						devName &&
						mainStore.config.enableNetCardMetric &&
						mainStore.config.netCardMetricValue &&
						mainStore.config.netCardMetricValue >= 1 &&
						mainStore.config.netCardMetricValue <= 9999
					) {
						try {
							const output = await Command.create(
								"netsh",
								["interface", "ipv4", "set", "interface", devName, "metric=", `${mainStore.config.netCardMetricValue}`],
								{
									encoding: "gb2312"
								}
							).execute();
						} catch (err) {
							console.error(err);
							ElMessage.error("跃点设置失败");
						}
					}
				}
				const logArr = data.log.split("\n");
				const start = logArr.length > 1000 ? logArr.length - 1000 : 0;
				data.log = logArr.slice(start).join("\n");
				data.log += (event.payload as string) + "\n";
			});
			this.unListenOutPut = unListen;
		},
		async listenThreadId() {
			const unListen = await listen("thread-id", event => {
				if (event.payload) {
					this.thread_id = event.payload;
				}
			});
			this.unListenThreadId = unListen;
		},
		async listenConfigStart() {
			const unListen = await listen("config", event => {
				// console.error("config", event.payload);
				const ipv4 = config.ipv4;
				mainStore.$patch(event.payload as any);
				config.ipv4 = ipv4;
			});
			this.unListenConfigStart = unListen;
		}
	};

	const unListenAll = async () => {
		listenObj.unListenOutPut && (await listenObj.unListenOutPut());
		await invoke("stop_command", { child_id: listenObj.thread_id || 0 });
		listenObj.thread_id = null;
	};

	const getCoreVersion = async (isClick = false) => {
		const coreVersion = await invoke("get_core_version");
		if (!coreVersion && isClick) {
			ElMessage.error("获取失败");
		}
		data.coreVersion = (coreVersion as string).replace("easytier-core ", "");
	};

	const checkUpdate = async () => {
		// const latestVersionFileName = data.releaseList?.[0]?.[0]?.[1] as string;
		let [downloadUrl, versionFileName] = (coreManagementData.data as string).split("<>");
		if (mainStore.githubFastUrl) {
			let fastUrl = mainStore.githubFastUrl.replace(/\\/g, "/").replace(/\/{2,}/g, "/");
			if (!fastUrl.endsWith("/")) fastUrl += "/";
			downloadUrl = fastUrl + downloadUrl;
		}
		if (versionFileName) {
			// console.error(latestVersionFileName, /\-v(\d+\.\d+\.\d+)/g.exec(latestVersionFileName));
			const version = /\-v(\d+\.\d+\.\d+)/g.exec(versionFileName)?.[1];
			const currentVersion = /(\d+\.\d+\.\d+)/g.exec(data.coreVersion || "")?.[1];
			if (version && currentVersion != version) {
				// console.error({ currentVersion, latestVersion });
				ElMessage.success(`更新 -> ${version}`);
				return [true, downloadUrl, versionFileName];
			} else if (currentVersion === version) {
				ElMessage.success(`当前版本无需安装`);
				return [false, null, null];
			} else {
				ElMessage.error(`获取版本失败`);
				return [false, null, null];
			}
		} else {
			ElMessage.error(`获取版本失败`);
		}
		return [false, null, null];
	};

	const handleCoreManagement = async () => {
		coreManagementData.visible = true;
		await getReleaseList();
	};

	const handleInstallCore = async () => {
		try {
			if (!coreManagementData.data) {
				return ElMessage.error("请选择一个内核");
			}
			data.update = true;
			await getCoreVersion();
			const [isNeedUpdate, downloadUrl, latestVersionFileName] = await checkUpdate();

			if (isNeedUpdate) {
				await reset();
				// console.error(downloadUrl);
				await invoke("download_easytier_zip", { download_url: downloadUrl, file_name: latestVersionFileName });
			}
			await getCoreVersion();
		} catch (err) {
			console.error(err);
		} finally {
			data.update = false;
		}
	};

	const getReleaseList = async () => {
		coreManagementData.loading = true;
		const list = await invoke<string[][][]>("fetch_easytier_list");
		data.releaseList = [
			...list.flat().filter(el => {
				// console.log(el, el[0], el[2]);
				return el && el[0] && el[2] && !el[2].includes("gui");
			})
		];
		coreManagementData.loading = false;
		// console.log(data.releaseList);
	};

	const handleAutoStart = async () => {
		let is_enable = await tauriAutoStart.isEnabled();
		if (!config.autoStart && !is_enable) {
			try {
				await tauriAutoStart.enable();
			} catch (err) {
				ElMessage.error(`开机自启失败`);
			}
		} else {
			try {
				await tauriAutoStart.disable();
			} catch (err) {
				// ElMessage.error(`取消自启失败`);
			}
		}
		is_enable = await tauriAutoStart.isEnabled();
		if (!is_enable) {
			config.autoStart = false;
		} else {
			config.autoStart = true;
		}
	};

	const initAutoStart = async () => {
		try {
			const is_enable = await tauriAutoStart.isEnabled();
			if (!is_enable) {
				config.autoStart = false;
			} else {
				config.autoStart = true;
			}
		} catch (err) {
			config.autoStart = false;
		}
	};

	const handleAutoStartByTask = async () => {
		await invoke("spawn_autostart", { enabled: !config.autoStart });
		const is_enable_by_task = (await invoke("autostart_is_enabled")) as boolean;
		config.autoStart = is_enable_by_task;
	};

	const compatibleInitAutoStart = async () => {
		try {
			const is_enable = await tauriAutoStart.isEnabled();
			if (is_enable) {
				await invoke("spawn_autostart", { enabled: true });
				await tauriAutoStart.disable();
			}
			let is_enable_by_task = (await invoke("autostart_is_enabled")) as boolean;
			if (is_enable_by_task) {
				// 每次打开Exe重新加载一次开机自启，因为可能路径变了
				await invoke("spawn_autostart", { enabled: false });
				await invoke("spawn_autostart", { enabled: true });
			}
			is_enable_by_task = (await invoke("autostart_is_enabled")) as boolean;
			config.autoStart = is_enable_by_task;
		} catch (err) {
			console.error(err);
			await invoke("spawn_autostart", { enabled: false });
			const is_enable_by_task = (await invoke("autostart_is_enabled")) as boolean;
			config.autoStart = is_enable_by_task;
		}
	};

	const initConnectAfterStart = async () => {
		if (config.connectAfterStart && data.coreVersion) {
			await reset();
			await handleConnection();
		}
	};

	const initGuiJson = async () => {
		const path = import.meta.env.VITE_CONFIG_FILE_NAME;
		const isExists = await exists(path, { baseDir: BaseDirectory.Resource });
		if (isExists) {
			const guiJsonStr = await readTextFile(path, { baseDir: BaseDirectory.Resource });
			if (guiJsonStr) {
				try {
					const regex = /^(\s*\/\/.*)/gm;
					const regex2 = /(,?)\s*\/\/.*(?=\n|$|\r\n)/gm;
					const resultStr = guiJsonStr.replace(regex, "").replace(regex2, "$1");
					const guiJson = JSON.parse(resultStr);
					let saveServerUrl = "";
					if (guiJson.serverUrl) {
						if (Array.isArray(guiJson.serverUrl)) {
							mainStore.basePeers = uniq([...guiJson.serverUrl, ...mainStore.basePeers]);
						}
						if (typeof guiJson.serverUrl === "string") {
							mainStore.basePeers = uniq([...guiJson.serverUrl.split(","), ...mainStore.basePeers]);
						}
						saveServerUrl = mainStore.basePeers[0] || "";
					} else {
						saveServerUrl = mainStore.basePeers.length > 0 ? mainStore.basePeers[0] || "" : "";
					}
					data.configJsonSeverUrl = guiJson.serverUrl;
					mainStore.$patch({
						config: {
							...mainStore.config,
							...guiJson,
							serverUrl: saveServerUrl
						}
					});
					if (mainStore.createConfigInEasytier) {
						await updateConfigJson(data.configJsonSeverUrl);
					}
				} catch (err) {
					console.error(err);
					ElMessage.error(`config.json格式错误`);
				} finally {
					mainStore.$patch({
						createConfigInEasytier: true // 发现本地存在config.json 默认启用该功能
					});
				}
			}
		}
		const b = bounce(600);
		mainStore.$subscribe((...a) => {
			if (mainStore.createConfigInEasytier) {
				b(async () => {
					await updateConfigJson(data.configJsonSeverUrl);
				});
			}
		});
	};

	let logsTimer: NodeJS.Timeout | null = null;

	onMounted(async () => {
		// await handleUpdateCore();  //默认不自动更新
		await initGuiJson();
		await compatibleInitAutoStart();
		// await initAutoStart();
		await initStartWinIpBroadcast();
		await getCoreVersion();
		await listenObj.listenThreadId();
		await listenObj.listenConfigStart();
		await initConfigDir();
		await initConnectAfterStart();
		closePrevent();
	});

	onBeforeUnmount(() => {
		unListenAll();
		listenObj.unListenReleaseList && listenObj.unListenReleaseList();
		listenObj.unListenConfigStart && listenObj.unListenConfigStart();
		logsTimer && clearInterval(logsTimer);
	});

	const getArgs = async () => {
		// console.error(config.proxyNetworks);
		const args = [];
		if (mainStore.configStartEnable) {
			if (mainStore.configPath) {
				const isExists = await exists(mainStore.configPath, { baseDir: BaseDirectory.Resource });
				if (isExists) {
					const resourceDir = await getResourceDir();
					const configPath = await join(resourceDir, mainStore.configPath);
					args.push("-c", configPath);
					return args;
				} else {
					const appWindow = getCurrentWindow();
					const isVisible = await appWindow.isVisible();
					if (!isVisible) {
						await appWindow.show();
						await appWindow.setFocus();
					}
					mainStore.configPath = "";
					ElMessage.error(`配置文件不存在 请重新选择`);
					await handleStartCommand("toml"); // 配置文件不存在，显示配置文件弹窗
					return [];
				}
			} else {
				ElMessage.warning(`配置文件不存在 请选择`);
				await handleStartCommand("toml"); // 配置文件不存在，显示配置文件弹窗
				return [];
			}
		}
		if (config.dhcp) {
			args.push("-d");
		}
		if (config.hostname) {
			args.push("--hostname", config.hostname);
		}
		if (config.networkName) {
			args.push("--network-name", config.networkName);
		}
		if (config.networkPassword) {
			args.push("--network-secret", config.networkPassword);
		}
		if (config.ipv4) {
			args.push("--ipv4", config.ipv4);
		}
		if (config.serverUrl) {
			const formatUrl = config.serverUrl.replace(/\\/g, "/");
			args.push("--peers", ...config.protocol.map(protocol => `${protocol}://${formatUrl}`));
		}
		if (config.disbleP2p) {
			args.push("--disable-p2p");
		}
		if (config.disableIpv6) {
			args.push("--disable-ipv6");
		}
		if (config.disbleListenner) {
			args.push("--no-listener");
		}
		if (mainStore.cidrEnable && config.proxyNetworks) {
			// console.error(config.proxyNetworks);
			const reg = /\d+\.\d+\.\d+\.\d+\/\d+/g;
			const formatProxyNetworks = config.proxyNetworks
				.split("\n")
				.map(item => item.trim())
				.filter(item => item && reg.test(item));
			args.push("--proxy-networks", ...formatProxyNetworks);
			config.proxyNetworks = formatProxyNetworks.join("\n");
		}
		if (config.disableEncryption) {
			args.push("--disable-encryption");
		}
		if (config.multiThread) {
			args.push("--multi-thread");
		}
		if (config.enablExitNode) {
			args.push("--enable-exit-node");
		}
		if (config.noTun) {
			args.push("--no-tun");
		}
		if (config.latencyfirst) {
			args.push("--latency-first");
		}
		if (config.useSmoltcp) {
			args.push("--use-smoltcp");
		}
		if (config.disableUdpHolePunching) {
			args.push("--disable-udp-hole-punching");
		}
		if (config.relayAllPeerrpc) {
			args.push("--relay-all-peer-rpc");
		}
		if (config.saveErrorLog) {
			args.push("--file-log-level", config.logLevel, "--file-log-dir", import.meta.env.VITE_LOG_PATH);
		}
		if (config.devName && config.devNameValue) {
			args.push("--dev-name", config.devNameValue);
		}
		return args;
	};

	const reset = async () => {
		data.isStart = false;
		data.isSuccessGetIp = false;
		if (config.dhcp) {
			config.ipv4 = "";
		}
		const memberDialog = await getAllWebviewWindows();
		const memberDialogs = memberDialog.filter(item => item.label === "member");
		if (memberDialogs && memberDialogs.length > 0) {
			data.memberVisible = false;
			for (const memberDialog of memberDialogs) {
				const isVisible = await memberDialog.isVisible();
				if (isVisible) {
					await memberDialog.destroy();
				}
			}
		}
		await setTrayRunState(tray, false);
		await setTrayTooltip(tray);
		await unListenAll();
	};

	const handleConnection = async () => {
		if (data.isStart) {
			await reset();
		} else {
			await setTrayTooltip(tray, "请求联机中...");
			const args = await getArgs();
			if (!args || args.length <= 0) {
				await reset();
				return;
			}
			if (args[0] === "-c") {
				ElMessage.warning({
					message: "使用配置文件中.",
					duration: 5000
				});
			}
			data.log = ""; //清空日志
			data.startLoading = true;
			await unListenAll();
			await listenObj.listenOutput();

			await invoke("run_command", {
				args
			});
		}
	};

	//configStart
	const handleStartCommand = async (command: string | number | object) => {
		if (command === "toml") {
			configStart.visible = true;
			const path = import.meta.env.VITE_CONFIG_PATH;
			const isExists = await exists(path, { baseDir: BaseDirectory.Resource });
			if (isExists) {
				const entries = await readDir(path, { baseDir: BaseDirectory.Resource });
				configStart.list = entries
					.filter(item => item.isFile)
					.map(item => ({
						name: item.name,
						path: `${path}${item.name}`
					})) as any;
			} else {
				mainStore.configStartEnable = false;
				mainStore.configPath = "";
				try {
					await mkdir(path, { baseDir: BaseDirectory.Resource });
				} catch (err) {}
			}
		}
		if (command === "share_config") {
			try {
				const {
					proxyNetworks,
					autoStart,
					connectAfterStart,
					multiThread,
					hostname,
					enablExitNode,
					useSmoltcp,
					saveErrorLog,
					logLevel,
					ipv4,
					...otherConfig
				} = mainStore.config;
				const WT = btoa(encodeURIComponent(JSON.stringify({ config: otherConfig })));
				await writeText(WT);
				ElMessage.success("配置已复制");
			} catch (err) {
				console.error(err);
				ElMessage.error("分享失败");
			}
		}
		if (command === "import_config") {
			importConfigData.data = "";
			importConfigData.visible = true;
		}
	};

	const handleStartImport = async () => {
		try {
			await ElMessageBox.confirm("确定导入?", "提示", {
				confirmButtonText: "确定",
				cancelButtonText: "取消"
			});
			const payload = JSON.parse(decodeURIComponent(atob(importConfigData.data)));
			mainStore.$patch({
				config: {
					...mainStore.config,
					...payload.config
				}
			});
			ElMessage.success("导入成功");
			importConfigData.visible = false;
			mainStore.basePeers = uniq([config.serverUrl, ...mainStore.basePeers]);
		} catch (err) {
			console.error(err);
			if (err !== "cancel") {
				ElMessage.error("导入失败");
			}
		}
	};

	const initConfigDir = async () => {
		const path = import.meta.env.VITE_CONFIG_PATH;
		const isExists = await exists(path, { baseDir: BaseDirectory.Resource });
		if (!isExists) {
			mainStore.configStartEnable = false;
			mainStore.configPath = "";
			try {
				await mkdir(path, { baseDir: BaseDirectory.Resource });
			} catch (err) {
				console.error(err);
			}
		}
	};

	const openConfigDir = async () => {
		const resourceDir = await getResourceDir();
		const configPath = await join(resourceDir, import.meta.env.VITE_CONFIG_PATH);
		// console.error(configPath);
		await Command.create("explorer", [configPath]).execute();
	};

	const handleShowMemberDialog = async () => {
		if (!data.isStart) {
			return ElMessage.warning("请先开始联机");
		}
		// if (!mainStore.config.ipv4) {
		// 	return ElMessage.warning("请等待获取IP");
		// }

		await etWindows(
			"member",
			{
				title: "成员列表",
				width: 875,
				height: 380,
				url: "#/member"
			},
			() => {
				data.memberVisible = true;
			},
			() => {
				data.memberVisible = false;
			}
		);
	};

	const handleShowLogDialog = async () => {
		await etWindows(
			"log",
			{
				title: "联机日志",
				width: 600,
				height: 380,
				resizable: false,
				url: "#/log"
			},
			(_, appWindow) => {
				data.logVisible = true;
				logsTimer && clearInterval(logsTimer);
				logsTimer = setInterval(() => {
					appWindow.emitTo("log", "logs", data.log);
				}, 600);
			},
			() => {
				data.logVisible = false;
			}
		);
	};

	const handleShowCidrDialog = async () => {
		await etWindows(
			"cidr",
			{
				title: "子网代理",
				width: 600,
				height: 380,
				resizable: false,
				url: "#/cidr"
			},
			(_, appWindow) => {
				data.cidrVisible = true;
			},
			() => {
				data.cidrVisible = false;
			}
		);
	};

	const handleShowAdvanceDialog = async () => {
		await etWindows(
			"advance",
			{
				title: "高级选项",
				width: 600,
				height: 380,
				resizable: false,
				url: "#/advance"
			},
			(_, appWindow) => {
				data.advanceVisible = true;
			},
			() => {
				data.advanceVisible = false;
			}
		);
	};

	const handleShowToolDialog = async () => {
		await etWindows(
			"tool",
			{
				title: "增强工具",
				width: 460,
				height: 480,
				resizable: true,
				url: "#/tool"
			},
			(_, appWindow) => {
				data.toolVisible = true;
			},
			() => {
				data.toolVisible = false;
			}
		);
	};
</script>
