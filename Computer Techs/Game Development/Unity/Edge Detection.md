```C
Shader "Hidden/Edge Detection"
{
    Properties
    {
        _OutlineThickness ("Outline Thickness", Float) = 1
        _OutlineColor ("Outline Color", Color) = (0, 0, 0, 1)
        _NormalThreshold("Normal Threshold", Range(0,1)) = 0.1
        _DepthThreshold("Depth Threshold", Range(0,1)) = 0.01
        _LuminanceThreshold("Luminance Threshold", Range(0,1)) = 0.25
    }
    SubShader
    {
        Tags
        {
            "RenderType"="Opaque"
            "RenderPipeline"="UniversalPipeline"
        }

        HLSLINCLUDE
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
        #include "Packages/com.unity.render-pipelines.core/Runtime/Utilities/Blit.hlsl"
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareNormalsTexture.hlsl"
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl"
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareOpaqueTexture.hlsl"

        float4 _OutlineColor;
        float _OutlineThickness;
        float _NormalThreshold;
        float _DepthThreshold;
        float _LuminanceThreshold;

        float RobertsCross(float samples[4])
        {
            float difference1 = samples[0] - samples[3];
            float difference2 = samples[1] - samples[2];
            return sqrt(difference1 * difference1 + difference2 * difference2);
        }

        float RobertsCross(float3 samples[4])
        {
            float3 difference1 = samples[0] - samples[3];
            float3 difference2 = samples[1] - samples[2];
            return sqrt(dot(difference1, difference1) + dot(difference2, difference2));
        }

        float SampleSceneLuminance(float2 uv)
        {
            float3 color = SampleSceneColor(uv);
            return 0.2989 * color.r + 0.5870 * color.g + 0.1140 * color.b;
        }
        ENDHLSL

        Pass
        {
            ZWrite Off
            Cull Off
            Blend SrcAlpha OneMinusSrcAlpha

            HLSLPROGRAM
            #pragma vertex Vert
            #pragma fragment OutlineFrag

            half4 OutlineFrag(Varyings IN) : SV_TARGET
            {
                float2 uv = IN.texcoord;
                float2 texelSize = float2(1 / _ScreenParams.x, 1 / _ScreenParams.y);

                const float halfWidthF = floor(_OutlineThickness * 0.5);
                const float halfWidthC = ceil(_OutlineThickness * 0.5);

                float2 uvs[4];
                uvs[0] = uv + float2(halfWidthF, halfWidthC) * texelSize * half2(-1, 1);
                uvs[1] = uv + float2(halfWidthC, halfWidthC) * texelSize * half2(1, 1);
                uvs[2] = uv + float2(halfWidthF, halfWidthF) * texelSize * half2(-1, -1);
                uvs[3] = uv + float2(halfWidthC, halfWidthF) * texelSize * half2(1, -1);

                float3 normalSamples[4];
                float depthSamples[4], luminanceSamples[4];
                for (int i = 0; i < 4; i++)
                {
                    normalSamples[i] = SampleSceneNormals(uvs[i]) * 0.5 + 0.5;
                    depthSamples[i] = SampleSceneDepth(uvs[i]);
                    luminanceSamples[i] = SampleSceneLuminance(uvs[i]);
                }

                float normalDetection = RobertsCross(normalSamples);
                float depthDetection = RobertsCross(depthSamples);
                float luminanceDetection = RobertsCross(luminanceSamples);

                normalDetection = normalDetection > _NormalThreshold ? 1 : 0;
                depthDetection = depthDetection > _DepthThreshold ? 1 : 0;
                luminanceDetection = luminanceDetection > _LuminanceThreshold ? 1 : 0;
                
                float edge = max(max(normalDetection, depthDetection), luminanceDetection);
                return edge * _OutlineColor;
            }
            ENDHLSL
        }
    }
}
```

```CS
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;
using UnityEngine.Rendering.RenderGraphModule;

public class EdgeDetection : ScriptableRendererFeature
{
    private class EdgeDetectionPass : ScriptableRenderPass
    {
        private class PassData
        {
        }
        
        private static readonly int OutlineColorProperty = Shader.PropertyToID("_OutlineColor");
        private static readonly int OutlineThicknessProperty = Shader.PropertyToID("_OutlineThickness");
        private static readonly int NormalThresholdProperty = Shader.PropertyToID("_NormalThreshold");
        private static readonly int DepthThresholdProperty = Shader.PropertyToID("_DepthThreshold");
        private static readonly int LuminanceThresholdProperty = Shader.PropertyToID("_LuminanceThreshold");

        private Material _material;

        public EdgeDetectionPass()
        {
            profilingSampler = new ProfilingSampler(nameof(EdgeDetectionPass));
        }

        public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
        {
            var resourceData = frameData.Get<UniversalResourceData>();

            using var builder = renderGraph.AddRasterRenderPass<PassData>("Edge Detection", out _);
            builder.SetRenderAttachment(resourceData.activeColorTexture, 0);
            builder.UseAllGlobalTextures(true);
            builder.AllowPassCulling(false);
            builder.SetRenderFunc<PassData>((_, context) => { Blitter.BlitTexture(context.cmd, Vector2.one, _material, 0); });
        }

        public void Setup(ref Settings edgeDetectionSettings, ref Material edgeDetectionMaterial)
        {
            _material = edgeDetectionMaterial;
            renderPassEvent = edgeDetectionSettings.renderPassEvent;

            _material.SetColor(OutlineColorProperty, edgeDetectionSettings.outlineColor);
            _material.SetFloat(OutlineThicknessProperty, edgeDetectionSettings.outlineThickness);
            _material.SetFloat(NormalThresholdProperty, edgeDetectionSettings.normalThreshold);
            _material.SetFloat(DepthThresholdProperty, edgeDetectionSettings.depthThreshold);
            _material.SetFloat(LuminanceThresholdProperty, edgeDetectionSettings.luminanceThreshold);
        }
    }

    [System.Serializable]
    public class Settings
    {
        public RenderPassEvent renderPassEvent = RenderPassEvent.AfterRenderingTransparents;
        public Color outlineColor = Color.black;
        [Range(0f, 15f)]public float outlineThickness = 1f;
        [Range(0f, 1f)] public float normalThreshold = 0.1f;
        [Range(0f, 1f)] public float depthThreshold = 0.01f;
        [Range(0f, 1f)] public float luminanceThreshold = 0.25f;
    }

    [SerializeField] private Settings edgeDetectionSettings;
    private Material _edgeDetectionMaterial;
    private EdgeDetectionPass _edgeDetectionPass;

    public override void Create()
    {
        _edgeDetectionPass ??= new EdgeDetectionPass();
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        if (renderingData.cameraData.cameraType == CameraType.Preview || renderingData.cameraData.cameraType == CameraType.Reflection || 
            UniversalRenderer.IsOffscreenDepthTexture(ref renderingData.cameraData))
            return;

        if (_edgeDetectionMaterial == null)
        {
            _edgeDetectionMaterial = CoreUtils.CreateEngineMaterial(Shader.Find("Hidden/Edge Detection"));
            if (_edgeDetectionMaterial == null)
            {
                Debug.LogError($"Can't find EdgeDetection Material");
                return;
            }
        }
        
        _edgeDetectionPass.ConfigureInput(ScriptableRenderPassInput.Normal | ScriptableRenderPassInput.Depth | ScriptableRenderPassInput.Color);
        _edgeDetectionPass.Setup(ref edgeDetectionSettings, ref _edgeDetectionMaterial);
        _edgeDetectionPass.requiresIntermediateTexture = true;
        
        renderer.EnqueuePass(_edgeDetectionPass);
    }

    protected override void Dispose(bool disposing)
    {
        _edgeDetectionPass = null;
        CoreUtils.Destroy(_edgeDetectionMaterial);
    }
}
```