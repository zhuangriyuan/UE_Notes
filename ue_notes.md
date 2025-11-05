# Unreal

## Level搭建
__Directional Light__
> intensity 0.8

__Exponential Height Fog__ 
> 颜色默认为黑所以没效果
> __Fog Density__ 1.0
> __Start Distance__ 调整

__PostProcess Volume__
> __Unbound__，Exposure中 __MinEV100__ 和 __MaxEV100__ 设置为1，__Exposure Compensation__ 为0

__Sky Light__
> intensity 3, 捕获天空光照反射

## Types
1. __Opaque（不透明）__
完全不透明，没有透明通道（Alpha 会被忽略）
渲染最快，Z-Buffer 深度写入最准确
__无法做半透明或透光效果__
> 适用场景：
> 固体模型、硬表面物体（石头、金属、墙壁）

2. __Masked（遮罩）__
通过 Opacity Mask（遮罩通道） 控制某像素是否可见（0 = 全透明，1 = 完全不透明，中间值会直接被二选一）
__支持硬边透明，但不能渐变透明__
渲染速度几乎接近 Opaque，因为深度写入完整
> 树叶、栅栏、铁丝网（形状不规则但需要硬边）

3. __Translucent（半透明）__
支持渐变透明（Opacity 0~1）
光可以穿过，能做玻璃、水、烟雾等效果
渲染成本较高
> 适用场景：
> 烟雾、雾气、玻璃、水面
> 柔和的粒子特效（爆炸火焰外围、魔法光晕）

4. __Additive（叠加）__
类似 Translucent，但透明像素会被加亮，而不是和背景平均混合
没有真正的“实心”部分，越亮的地方越明显
适合用发光感替代颜色混合
> 适用场景：
> 火焰、魔法、能量光效
> 爆炸火花、科幻 UI 发光线条
> 光照贴花、镜头炫光（Lens Flare）

### Material

Camera Vector已经normalize了
Camera Position - World Position没有normalize

## Material Nodes
__StaticSwitchParameter__
> 用于切换/开关

__VectorLength__
> 计算distance

SceneTexture
> Postprocess Material

__Flatten Normal__
> 加强normal

__Component Mask__
> vector to float

__Make Float234__
> float to vector

__Component Mask__
> vector to float

__One minus__
> complement

__Vertex Normal WS__
> 获取法线

__Transform Vector__ (Transform)
> 将vector转换到world space (Tagnent to world space)

__Camera Position__
> 获取相机位置

__Absolute World Position__
> 获取绝对世界坐标轴

__Location Position__
> local pos

__Saturate__
> clamp value to 0-1 at no cost

__Desaturation__
> RGB to Mono

__Min Max Clamp Saturated__
> remap values

__Smooth Steps__
> 做gradient

__LinearGradient__
> 黑白过度

__Sphere Mask__
> create a sphere mask, input A (TexCoord), input B (0.5, 0.5) center at uv0.5, radius 0.5, hardness 0

__GeneratedBand__
> 也是mask

__Frac__
> 15.25 return 0.25

__Fmod__
> %

## Niagara

 __Translucent__ - 会显示黑色
 __Additive__ - 黑色会变透明

#### Expression
Particles.RibbonID
> 用于Ribbon

__Dynamic Paramater__
> 可以和material属性联动

__Subuv Animation__
> 在particle update中使用，不止可以播放subuv，也可以随机取一张instance到粒子上，改成random - spawn only

__Event__
1. Generate Collision Event
2. Generate Death Event
    > 需要collision模块，然后kill particles选择bool (has collided),最后generate death event
    > 子emitter同理，需要event handler和receive模块
3. Generate Location Event （Trail）
    > emitter不需要生成粒子，靠的是event handler来生成粒子
    > 父发射器Properties中勾选 __persistent id__
    > 子发射器在stage中+ __Event Handler__，在source中选中 __locationEvent__，__execution mode为spawned particles__，__spawn number__ 为1.最后加上 __receive location event模块__
    > 也可以在父发射器中调节 __send rate__ 来调节粒子生成数量

__Spawn Particles From Other Emitter__
__Sample Particles From Other Emitter__
> 做trail除了location event也可以用这两个在子发射器上，Sample中也需要讲id勾选

__System Location__
> offset position, Initialize Partizle中也有offset功能

__Set Parameters__
> Set (Particles) Velocity -> BeamSplineTangent

__Sprite Facing and Alignment__
> 也需要在sprite render中设置alignment和facing mode 

__Initial Mesh Orientation__

__Kill Particles In Volume__

__Sprite Rotation Rate__
> subuv选择素材后，可以用这个控制旋转
> 可以设置为bool，用collisionResting这个属性控制bool，这样可以让落地的旋转消失
> 需要在collision中开启Max Number of Collisions

__Static Mesh Location__
> Preview Mesh中选择网格体，可以用模型发射粒子

__Skeletal Mesh Location__
> 如果想要在特定骨骼发射粒子，可以在filtered bones中选择，然后在bone sampling mode中选择Random（Filtered Bones）

__Initial Mesh Orientation__
> 如果velocity align后朝向还是不对，可以用这个改一下初始旋转，mesh orientation mode改为system，下面调整角度

__用贴图发射粒子__
1. Spawn Particles in Grid 
2. Sample Texture(uv中Make Vector 2D -> Make Float from Vector -> Normalized Array Location)

__Ribbon Renderer更改分段数__
> 在Ribbon Tessellation中调整Custom mode中Max Tessellation Factor为1可以让jitter的ribbon转折更清晰

__调整Ribbon的scale__
> 可以使用RibbonLinkOrder来代替NormalizedAge，类似houdini中的id