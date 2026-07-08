## *1/3)* A potential fix for Visual Studio 2026 Community Edition with enabling GLSL shaders with syntax highlighting ONLY.
You do not have to download anything from Visual Studio Marketplace. This is done without any extensions, just vanilla ***Visual Studio 2026***.<br>
**If TL;DR, skip to bottom!**

This is for anyone who's had frustrating experiences with context highlightning syntax *and* multi-staged shaders combined into a *single shader file*:
  - vs -> fs
  - vs -> gs -> fs
  - vs -> tcs -> tes -> fs
  - vs -> tcs -> tes -> gs -> fs

# **&#8594;**The problem is [shaderlab.json](shaderlab.json)**&#8592;**
For Visual Studio 2026, this file is located here:
`C:\Program Files\Microsoft Visual Studio\18\Community\Common7\IDE\CommonExtensions\Microsoft\TextMate\Starterkit\Extensions\shaderlab\syntaxes\shaderlab.json`

You may have noticed this from back as early as **Visual Studio 2019**, possibly even **Visual Studio 2017**. The first extensionless file called ***shader*** will 
show default syntax highlighting, yet for 'shader2', 'shader3', 'shader4', or any other file you create will not.

A partial workaround to this problem was: Tools -> Options -> Text Editor -> File Extensions -> adding your extension `.glsl` and selecting
the drop down menu such as Visual Studio C++. You would get the syntax highlighting *plus* error correction. However, this error correction becomes 
problematic because of certain parsing tokens when separating each distinct shader which needs even more workarounds. For instance:
```
#vertex shader
#version 460 core
... code here ...

#fragment shader
#version 460 core
```
These would show up as error squiggly lines on line 2 `#vertex shader` & line 3 `#version 460 core`. You could add `//` marks before them
but then you would have to keep track of all your shader files and so forth. At this point, its just another fix of a fix of a fix.

This solution allows syntax highlighting to all shader files ***only***, with no error validation.

Open shaderlab.json with any text editor and there is a single extensionless `shader` file that works by default with syntax highlighting.
<details>
<img width="1290" height="225" alt="shaderFile" src="https://github.com/user-attachments/assets/a1ffc226-a4e5-4341-a6dd-6adb9d1a89ee" />
</details>

The solution? Simply add any filename you wish here and Visual Studio will include the default highlighting syntax to all your other GLSL shader files as well!
<details>
<img width="978" height="511" alt="multiples" src="https://github.com/user-attachments/assets/e2eaa340-da13-4fa2-930f-c641a0248187" />
</details>
If you want to stop here, save the shaderlab.json file and restart Visual Studio 2026. Remove all your shader files & reload them again. You now have all your GLSL files 
with syntax highlighting without validation errors.

&nbsp;

If you try adding in context keywords for GLSL such as `gl_Position, _MainTex, COLOR, Albedo, worldPos, etc..`, you'll notice these will ***NOT*** light up. 
There is a bug in the matching conditional statement that silently errors out and skips context highlighting. It's possible this has been unfixed 
for 10+ years with Visual Studio IDE.  :bug::bug:

## *2/3)* To allow all GLSL context keywords to light up, copy the rest of this portion; Or add/remove as much or as little as you want.
```
"patterns": [
    {
      "match": "\\b([0-9]+\\.?[0-9]*)\\b",
      "name": "constant.numeric.shaderlab"
    },
    {
      "match": "\\b(Shader|Properties|Category|SubShader|Tags|Pass)\\b",
      "name": "support.class.shaderlab"
    },
    {
      "match": "\\b(CGPROGRAM|ENDCG)\\b",
      "name": "support.class.shaderlab"
    },
    {
      "match": "\\b(Dependency|Fallback)\\b",
      "name": "support.variable.shaderlab"
    },
    {
      "match": "^\\s*\\[(HideInInspector)\\]",
      "name": "keyword.shaderlab"
    },
    {
      "include": "#comments"
    },
    {
      "include": "#strings"
    },
    {
      "include": "#functions"
    },
    {
      "include": "#cg"
    }
  ],
  "repository": {
    "comments": {
      "patterns": [
        {
          "begin": "//",
          "end": "$",
          "name": "comment.line.double-slash.shaderlab"
        },
        {
          "begin": "/\\*",
          "end": "\\*/",
          "name": "comment.line.block.shaderlab"
        }
      ]
    },
    "strings": {
      "patterns": [
        {
          "begin": "\"",
          "end": "\"",
          "name": "string.quoted.double.shaderlab"
        }
      ]
    },
    "functions": {
      "patterns": [
        {
          "match": "(?x) (?: (?= \\s )  (?:(?<=else|new|return) | (?<!\\w)) (\\s+))?\n\t\t\t(\\b \n\t\t\t\t(?!(while|for|do|if|else|switch|catch|enumerate|return|sizeof|[cr]?iterate)\\s*\\()(?:(?!NS)[A-Za-z_][A-Za-z0-9_]*+\\b | :: )++\t\t\t\t  # actual name\n\t\t\t)\n\t\t\t \\s*(\\()",
          "captures": {
            "1": {
              "name": "punctuation.whitespace.function-call.leading.shaderlab"
            },
            "2": {
              "name": "support.function.any-method.shaderlab"
            },
            "3": {
              "name": "punctuation.definition.parameters.shaderlab"
            }
          },
          "name": "meta.function-call.shaderlab"
        }
      ]
    },
    "cg": {
      "patterns": [
        {
          "match": "^\\s*#\\s*(define|defined|elif|else|if|ifdef|ifndef|line|pragma|undef|version|extension|include|error)\\b",
          "name": "entity.name.class.shaderlab"
        },
        {
          "match": "\\.([rgba]{1,4}|[xyzw]{1,4}|[stpq]{1,4})\\b",
          "name": "entity.name.class.shaderlab"
        },
        {
          "match": "\\b(if|else|for|while|do|return|break|continue|switch|case|default|discard)\\b",
          "name": "string.shaderlab"
        },
        {
          "match": "\\b(const|extern|in|inline|inout|static|out|uniform|attribute|varying|layout|invariant|precision|highp|mediump|lowp|flat|smooth|noperspective|centroid|sample|patch|readonly|writeonly|coherent|volatile|restrict|shared|buffer|subroutine)\\b",
          "name": "string.shaderlab"
        },
        {
          "match": "\\b(gl_Position|gl_PointSize|gl_ClipDistance|gl_CullDistance|gl_FragCoord|gl_FrontFacing|gl_FragDepth|gl_PointCoord|gl_PrimitiveID|gl_SampleID|gl_SamplePosition|gl_SampleMaskIn|gl_SampleMask|gl_Layer|gl_ViewportIndex|gl_VertexID|gl_InstanceID|gl_InvocationID|gl_FragColor|gl_FragData|gl_GlobalInvocationID|gl_LocalInvocationID|gl_LocalInvocationIndex|gl_WorkGroupSize|gl_WorkGroupID|gl_NumWorkGroups|gl_PrimitiveIDIn|gl_PrimitiveIndicesNV|gl_TessCoord|gl_TessLevelOuter|gl_TessLevelInner|gl_PatchVerticesIn|gl_MaxVertexAttribs|gl_MaxCombinedTextureImageUnits|gl_MaxDrawBuffers|gl_MaxFragmentUniformComponents|gl_MaxTextureImageUnits|gl_MaxVaryingFloats|gl_MaxVertexUniformComponents|gl_MaxVertexTextureImageUnits|gl_DepthRange|gl_BaseVertex|gl_BaseInstance|gl_DrawID|gl_HelperInvocation|gl_SubgroupID|gl_SubgroupSize|gl_SubgroupInvocation|gl_BoundingBox|gl_ViewportMask|gl_MaxSamples|gl_MaxImageUnits|gl_MaxComputeWorkGroupCount|gl_MaxComputeWorkGroupSize|gl_MaxGeometryOutputVertices|gl_MaxPatchVertices|gl_MinProgramTexelOffset|gl_MaxProgramTexelOffset|gl_MaxClipDistances|gl_MaxCullDistances|gl_DepthRange|_MainTex|_Color|_Time|_SinTime|_CosTime|unity_DeltaTime|_ProjectionParams|_ScreenParams|viewDir|COLOR|screenPos|worldPos|worldRefl|worldNormal|Albedo|Normal|Emission|Specular|Gloss|Alpha|_Object2World|_World2Object|_WorldSpaceCameraPos|unity_Scale|_ObjectSpaceLightPos|appdata_base|appdata_tan|appdata_full|appdata_img|SurfaceOutput|UNITY_MATRIX_MVP|UNITY_MATRIX_MV|UNITY_MATRIX_V|UNITY_MATRIX_P|UNITY_MATRIX_VP|UNITY_MATRIX_T_MV|UNITY_MATRIX_IT_MV|UNITY_MATRIX_TEXTURE0|UNITY_MATRIX_TEXTURE1|UNITY_MATRIX_TEXTURE2|UNITY_MATRIX_TEXTURE3|void|struct|typedef|signed|unsigned|double|dvec[234]|dmat[234]|dmat[234]x[234]|float|vec[234]|mat[234]|mat[234]x[234]|half|fixed|long|int|ivec[234]|short|char|bool|bvec[234]|uint|uvec[234]|sampler[12]D|sampler3D|samplerCube|sampler[12]DShadow|samplerCubeShadow|sampler[12]DArray|sampler[12]DArrayShadow|sampler2DRect|sampler2DRectShadow|samplerBuffer|sampler2DMS|sampler2DMSArray|samplerCubeArray|samplerCubeArrayShadow|sampler1D|sampler1DShadow|sampler1DArray|sampler1DArrayShadow|isampler[123]D|isamplerCube|isampler[12]DArray|isampler2DRect|isampler2DMS|isampler2DMSArray|isamplerCubeArray|isampler1D|isampler1DArray|usampler[123]D|usamplerCube|usampler[12]DArray|usampler2DRect|usampler2DMS|usampler2DMSArray|usamplerCubeArray|usampler1D|usampler1DArray|image[123]D|imageCube|image[12]DArray|image2DRect|imageBuffer|image2DMS|image2DMSArray|imageCubeArray|image1D|image1DArray|iimage[123]D|iimageCube|uimage[123]D|uimageCube|atomic_uint)\\b",
          "name": "entity.name.class.shaderlab"
        },
        {
          "match": "\\b(radians|degrees|sin|cos|tan|asin|acos|atan|sinh|cosh|tanh|asinh|acosh|atanh|pow|exp|log|exp2|log2|sqrt|inversesqrt|abs|sign|floor|ceil|fract|mod|modf|min|max|clamp|mix|step|smoothstep|isnan|isinf|floatBitsToInt|floatBitsToUint|intBitsToFloat|uintBitsToFloat|fma|frexp|ldexp|packUnorm2x16|packUnorm4x8|packSnorm2x16|packSnorm4x8|unpackUnorm2x16|unpackUnorm4x8|unpackSnorm2x16|unpackSnorm4x8|packHalf2x16|unpackHalf2x16|packDouble2x32|unpackDouble2x32|length|distance|dot|cross|normalize|reflect|refract|faceforward|matrixCompMult|outerProduct|transpose|determinant|inverse|lessThan|lessThanEqual|greaterThan|greaterThanEqual|equal|notEqual|any|all|not|uaddCarry|usubBorrow|umulExtended|imulExtended|bitfieldExtract|bitfieldInsert|bitfieldReverse|bitCount|findLSB|findMSB|textureSize|textureQueryLod|textureQueryLevels|texture|textureProj|textureLod|textureOffset|texelFetch|texelFetchOffset|textureProjOffset|textureLodOffset|textureProjLod|textureProjLodOffset|textureGrad|textureGradOffset|textureProjGrad|textureProjGradOffset|textureGather|textureGatherOffset|textureGatherOffsets|textureSamples|atomicAdd|atomicMin|atomicMax|atomicAnd|atomicOr|atomicXor|atomicExchange|atomicCompSwap|atomicCounter|atomicCounterDecrement|atomicCounterIncrement|atomicCounterAdd|atomicCounterSubtract|atomicCounterMin|atomicCounterMax|atomicCounterAnd|atomicCounterOr|atomicCounterXor|atomicCounterExchange|atomicCounterCompSwap|imageSize|imageSamples|imageLoad|imageStore|imageAtomicAdd|imageAtomicMin|imageAtomicMax|imageAtomicAnd|imageAtomicOr|imageAtomicXor|imageAtomicExchange|imageAtomicCompSwap|dFdx|dFdy|dFdxFine|dFdyFine|dFdxCoarse|dFdyCoarse|fwidth|fwidthFine|fwidthCoarse|interpolateAtCentroid|interpolateAtSample|interpolateAtOffset|EmitStreamVertex|EndStreamPrimitive|EmitVertex|EndPrimitive|barrier|memoryBarrier|memoryBarrierAtomicCounter|memoryBarrierBuffer|memoryBarrierShared|memoryBarrierImage|groupMemoryBarrier)\\b",
          "name": "string.shaderlab"
        },
        {
          "match": "\\b(main)\\s*(?=\\()",
          "name": "entity.name.class.shaderlab"
        },
        {
          "match": "\\b(true|false)\\b",
          "name": "entity.name.class.shaderlab"
        }
      ]
    }
  }
}
```
Or just copy the entire contents inside [shaderlab.json](shaderlab.json) and paste it.

Here are the list of keywords in alphabetical order:

<details>
  
```
bool
bvec2
bvec3
bvec4
dmat2
dmat3
dmat4
dmat2x2
dmat2x3
dmat2x4
dmat3x2
dmat3x3
dmat3x4
dmat4x2
dmat4x3
dmat4x4
double
dvec2
dvec3
dvec4
float
int
ivec2
ivec3
ivec4
mat2
mat3
mat4
mat2x2
mat2x3
mat2x4
mat3x2
mat3x3
mat3x4
mat4x2
mat4x3
mat4x4
uint
uvec2
uvec3
uvec4
vec2
vec3
vec4
void
sampler1D
sampler1DArray
sampler1DArrayShadow
sampler1DShadow
sampler2D
sampler2DArray
sampler2DArrayShadow
sampler2DMS
sampler2DMSArray
sampler2DRect
sampler2DRectShadow
sampler2DShadow
sampler3D
samplerBuffer
samplerCube
samplerCubeShadow
isampler1D
isampler1DArray
isampler2D
isampler2DArray
isampler2DMS
isampler2DMSArray
isampler2DRect
isampler3D
isamplerBuffer
isamplerCube
usampler1D
usampler1DArray
usampler2D
usampler2DArray
usampler2DMS
usampler2DMSArray
usampler2DRect
usampler3D
usamplerBuffer
usamplerCube
image1D
image1DArray
image2D
image2DArray
image2DMS
image2DMSArray
image2DRect
image3D
imageBuffer
imageCube
iimage1D
iimage1DArray
iimage2D
iimage2DArray
iimage2DMS
iimage2DMSArray
iimage2DRect
iimage3D
iimageBuffer
iimageCube
uimage1D
uimage1DArray
uimage2D
uimage2DArray
uimage2DMS
uimage2DMSArray
uimage2DRect
uimage3D
uimageBuffer
uimageCube
attribute
buffer
centroid
coherent
const
flat
highp
in
inout
invariant
layout
lowp
mediump
noperspective
out
patch
precise
precision
readonly
restrict
sample
shared
smooth
uniform
varying
volatile
writeonly
abs
acos
asin
atan
acosh
asinh
atanh
ceil
clamp
cos
cosh
degrees
exp
exp2
floatBitsToInt
floatBitsToUint
floor
fract
intBitsToFloat
inversesqrt
isinf
isnan
log
log2
max
min
mix
mod
modf
pow
radians
round
roundEven
sign
sin
sinh
smoothstep
sqrt
step
tan
tanh
trunc
uintBitsToFloat
all
any
atomic_unit
atomicAdd
atomicAnd
atomicCompSwap
atomicCounter
atomicCounterIncrement
atomicCounterDecrement
atomicExchange
atomicMax
atomicMin
atomicOr
atomicXor
barrier
bitCount
bitfieldExtract
bitfieldInsert
bitfieldReverse
cross
determinant
distance
dot
EmitVertex
EndPrimitive
equal
faceforward
findLSB
findMSB
fma
frexp
greaterThan
greaterThanEqual
groupMemoryBarrier
imageAtomicAdd
imageAtomicAnd
imageAtomicCompSwap
imageAtomicExchange
imageAtomicMax
imageAtomicMin
imageAtomicOr
imageAtomicXor
imageLoad
imageSize
imageStore
imulExtended
inverse
ldexp
length
lessThan
lessThanEqual
matrixCompMult
memoryBarrier
memoryBarrierAtomicCounter
memoryBarrierBuffer
memoryBarrierImage
memoryBarrierShared
normalize
not
notEqual
outerProduct
packDouble2x32
packHalf2x16
packSnorm2x16
packSnorm4x8
packUnorm2x16
packUnorm4x8
reflect
refract
texelFetch
texelFetchOffset
texture
textureGather
textureGatherOffset
textureGatherOffsets
textureGrad
textureGradOffset
textureLod
textureLodOffset
textureOffset
textureProj
textureProjGrad
textureProjGradOffset
textureProjLod
textureProjLodOffset
textureProjOffset
textureQueryLevels
textureQueryLod
textureSize
transpose
uaddCarry
umulExtended
unpackDouble2x32
unpackHalf2x16
unpackSnorm2x16
unpackSnorm4x8
unpackUnorm2x16
unpackUnorm4x8
usubBorrow
gl_ClipDistance
gl_ClipVertex
gl_FragColor
gl_FragCoord
gl_FragData
gl_FragDepth
gl_FrontFacing
gl_GlobalInvocationID
gl_InstanceID
gl_InvocationID
gl_Layer
gl_LocalInvocationID
gl_LocalInvocationIndex
gl_NumSamples
gl_NumWorkGroups
gl_PatchVerticesIn
gl_PointCoord
gl_PointSize
gl_Position
gl_PrimitiveID
gl_PrimitiveIDIn
gl_SampleID
gl_SampleMaskIn
gl_SamplePosition
gl_TessCoord
gl_TessLevelInner
gl_TessLevelOuter
gl_VertexID
gl_ViewportIndex
gl_WorkGroupID
gl_WorkGroupSize
break
case
continue
default
discard
do
else
false
for
if
return
struct
subroutine
switch
true
while
```
</details>

## *3/3)* To change the color of your context keywords, you can replace the `"name"` section of the built-in scope names to any of the following and more.

The color output will be based on your current theme that is selected. It's approximately from a color pool of 5 colors but it should be sufficient enough.

  | Scope | Typical Color |
| :--- | :--- |
| **entity.name.class.shaderlab** | Teal/Green |
| **entity.name.function.shaderlab** | Teal/Green |
| **keyword.shaderlab.shaderlab** | Light Blue |
| **support.function.shaderlab** | Light Blue |
| **storage.type.shaderlab** | Light Blue |
| **markup.heading.shaderlab** | Brown/Orange |
| **string.shaderlab** | Brown/Orange |
| **string.quoted.shaderlab** | Brown/Orange |
| **constant.numeric.shaderlab** | Light Green |
| **variable.parameter.shaderlab** | Grey |
| **markup.underline.shaderlab** | Grey |
| **comment** | Green |

<br>
Now all your GLSL shader files will have context highlighting<i><b>(1/3)</b></i>, along with all GLSL context keywords<i><b>(2/3)</b></i>, and add extra colors to them<i><b>(3/3)</b></i>. Here is an example:
<br>

<br>
<img width="2040" height="1492" alt="shader" src="https://github.com/user-attachments/assets/dde80d36-9798-43ef-9e83-f0dc38a06b72" />

<br>
<br>

##&#8227;TL;DR: Here is the [shaderlab.json](shaderlab.json) file.

1. **Place it in:** `C:\Program Files\Microsoft Visual Studio\18\community\common7\IDE\CommonExtensions\Microsoft\TextMate\Starterkit\Extensions\shaderlab\syntaxes\`
2. **Restart** Visual Studio 2026.
3. **Remove** your shader files from your project. *(Don't delete them, just add them back in).*
4. **Open** your GLSL shader files.
5. GLSL shader files now have color highlights, *with* context keywords, and custom colors. <ins>***Finally!***</ins>
<br>
<br>

**&#8594;** <ins>**NOTE:**</ins> A **Visual Studio update** may overwrite shaderlab.json, so keep a backup of your edited version or a note to re-add your filenames after update.
