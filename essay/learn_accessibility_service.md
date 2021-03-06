# 无障碍服务研究

## AccessibilityService
无障碍服务类
* onAccessibilityEvent - 无障碍事件的回调方法
* onInterrupt - 访问中断的回调
* onServiceConnected - 服务连接的回调
* onGesture - 触摸手势回调
* onKeyEvent - 按键回调
* getWindows -返回当前窗口上所有的Window
* getWindowsOnAllDisplays -返回所有窗口上的Window
* getRootInActiveWindow -返回当前窗口的root Node
* disableSelf -关闭当前服务 
* getFingerprintGestureController 获取指纹手势的控制器
* dispatchGesture - 分发手势
* getSoftKeyboardController - 返回软键盘控制器，可用于查询和修改软键盘显示模式
* getSystemActions - 返回当前系统可用的无障碍Actions
* performGlobalAction -执行全局动作
* findFocus - 查找焦点选中的Node
* getServiceInfo -获取改服务的信息
* takeScreenshot -屏幕截图
* setAccessibilityFocusAppearance -设置辅助焦点的描边大小和颜色
## AccessibilityEvent
无障碍服务回调的事件类
## AccessibilityNodeInfo
无障碍服务的一个节点
* focusSearch - 在指定方向上搜索可以获取输入焦点的最近视图
* getChildCount - 返回节点count
* getChild - 根据索引获取子节点Node
* getActionList - 获取可以在节点执行的Actions
* getAvailableExtraData - 获取此节点可用的额外数据
* getMaxTextLength - 返回此节点的最大文本长度
* performAction - 在节点上执行操作
* findAccessibilityNodeInfosByText - 通过文本查找节点
* findAccessibilityNodeInfosByViewId - 通过viewId查找节点
* getWindow - 获取该节点所属的Window
* getParent - 获取父节点
* getBoundsInScreen - 获取屏幕坐标中的节点边界
* isCheckable - 该节点是否可被选中
* isChecked - 该节点是否被选中
* isFocused - 该节点是否被聚焦
* isVisibleToUser - 此节点是否对用户可见
* isAccessibilityFocused - 此节点是否可被聚焦
* isSelected - 此节点是否可被选中
* isClickable - 此节点是否可被点击
* isLongClickable - 此节点是否可被长按
* isEnabled - 此节点是否可用
* isPassword - 此节点是否未密码
* isScrollable - 是否可滚动
* isEditable - 是否可输入文字
* getDrawingOrder - 获取绘制顺序
* getCollectionInfo - 如果节点是集合，则获取集合信息。集合子始终是集合项
* isContentInvalid - 获取此节点的内容是否无效。例如，日期格式不正确。
* isContextClickable - 鼠标点击是否可用
* isMultiLine - 获取节点是否为多行可编辑文本
* canOpenPopup - 获取此节点是否打开弹出窗口或对话框
* isDismissable - 是否可以关闭该节点
* getPackageName - 获取包名
* getClassName - 获取类名
* getText - 获取此节点上的文字
* replaceClickableSpan - 替换节点上的文本
* getViewIdResourceName - 获取全路径的ViewId
* getInputType - 获取输入类型，可能用于EditText
* getExtras - 获取额外的数据信息