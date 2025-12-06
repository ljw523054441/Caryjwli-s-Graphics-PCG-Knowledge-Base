# GernsterWave
## 原版
```c++
float3 P = float3(0.0, 0.0, 0.0);
float3 B = float3(0.0, 1.0, 0.0);
float3 T = float3(1.0, 0.0, 0.0);

for (int i = 0; i < wavecount; i++)
{
    float step = (float) i / wavecount;

    float2 d = float2(frac(sin(dot(float2((float) i,2), float2(12.9898, 78.233)))) * 2 - 1, frac(sin(dot(float2((float) i * 2,2), float2(12.9898, 78.233)))) * 2 - 1);
    d = normalize(lerp(normalize(direction.xy), normalize(d), randomDirection));

    float wavelength = lerp(wavelengthMax, wavelengthMin, step);
    float steepness = lerp(wavesteepnessMax, wavesteepnessMin, step)/wavecount;

    float k = 2 * PI / wavelength;
    float g = 9.81f;
    float w = sqrt(g * k);
    float a = steepness / k;
    float2 wavevector = k * d;
    float value = dot(wavevector, worldpos.xy) - w * time * wavespeed;

    P.x += d.x * a * cos(value);
    P.y += d.y * a * cos(value);
    P.z += a * sin(value);

    T.x += d.x * d.x * k * a * -sin(value);
    T.y += d.x * d.y * k * a * -sin(value);
    T.z += d.x * k * a * cos(value);

    B.x += d.x * d.y * k * a * -sin(value);
    B.y += d.y * d.y * k * a * -sin(value);
    B.z += d.y * k * a * cos(value);
}

normal = normalize(cross(T,B));
return P;
```
## 带flowmap版
`direction.xy`为flowdir，`FlowmapMask`为flowmap生效的范围
```c++
float3 P = float3(0.0, 0.0, 0.0);
float3 B = float3(0.0, 1.0, 0.0);
float3 T = float3(1.0, 0.0, 0.0);

for (int i = 0; i < wavecount; i++)
{
    float step = (float) i / wavecount;

    float2 d = float2(frac(sin(dot(float2((float) i,2), float2(12.9898, 78.233)))) * 2 - 1, frac(sin(dot(float2((float) i * 2,2), float2(12.9898, 78.233)))) * 2 - 1);
    //d = normalize(lerp(normalize(direction.xy), normalize(d), randomDirection));
    d = normalize(d);

    float scale = (dot(d,direction.xy) + 1) * 0.5;//0-1 -> 
    scale = lerp(1, scale * ScaleMult, FlowmapMask);

    float wavelength = lerp(wavelengthMax, wavelengthMin, step);
    float steepness = lerp(wavesteepnessMax, wavesteepnessMin, step)/wavecount;

    float k = 2 * PI / wavelength;
    float g = 9.81f;
    float w = sqrt(g * k);
    float a = scale * steepness / k;
    float2 wavevector = k * d;
    float value = dot(wavevector, worldpos.xy) - w * time * wavespeed;

    P.x += d.x * a * cos(value);
    P.y += d.y * a * cos(value);
    P.z += a * sin(value);

    T.x += d.x * d.x * k * a * -sin(value);
    T.y += d.x * d.y * k * a * -sin(value);
    T.z += d.x * k * a * cos(value);

    B.x += d.x * d.y * k * a * -sin(value);
    B.y += d.y * d.y * k * a * -sin(value);
    B.z += d.y * k * a * cos(value);
}

normal = normalize(cross(T,B));
return P;
```

![[Pasted image 20250114210857.png]]

# FBMWave
## 原版
```C++
struct Functions{
    float3 fragmentFBM(float3 v) 
    {
		float f = FragmentFrequency;
		float a = FragmentAmplitude;
		float speed = FragmentInitialSpeed;
		float seed = FragmentSeed;
		float3 p = v;

		float h = 0.0f;
		float2 n = 0.0f;
				
		float amplitudeSum = 0.0f;

		for (int wi = 0; wi < FragmentWaveCount; ++wi) 
        {
			float2 d = normalize(float2(cos(seed), sin(seed)));

            //float2 d = float2(frac(sin(dot(float2((float) wi,2), float2(12.9898, 78.233)))) * 2 - 1, frac(sin(dot(float2((float) wi * 2,2), float2(12.9898, 78.233)))) * 2 - 1);
            //d = normalize(lerp(normalize(direction.xy), normalize(d), randomDirection));

			float x = dot(d, p.xy) * f + Time * speed;
			float wave = a * (exp(FragmentMaxPeak * sin(x) - FragmentPeakOffset) - 1.535);
            //float wave = a * FragmentMaxPeak * sin(x) - FragmentPeakOffset;
			float2 dw = f * d * (FragmentMaxPeak * wave * cos(x));
			
			h += wave;
			p.xy += -dw * a * FragmentDrag;
					
			n += dw;
					
			amplitudeSum += a;
			f *= FragmentFrequencyMult;
			a *= FragmentAmplitudeMult;
			speed *= FragmentSpeedRamp;
			seed += FragmentSeedIter;
		}
				
		float3 output = float3(h, n.x, n.y) / amplitudeSum;
		output.x *= FragmentHeight;

		return output;
    }
};

Functions func;

normal = float3(0.0f, 0.0f, 1.0f);
float height = 0.0f;
float3 fbm = func.fragmentFBM(InWorldPosition);

height = fbm.x;
normal.xy = -fbm.yz;
normal = normalize(normal);

return float3(0.0, 0.0, height);
```

## 带flowmap并且流速一致
`direction.xy`为flowdir，`FlowmapMask`为flowmap生效的范围
```c++
struct Functions{
    float3 fragmentFBM(float3 v) 
    {
		float f = FragmentFrequency;
		float a = FragmentAmplitude;
		float speed = FragmentInitialSpeed;
		float seed = FragmentSeed;
		float3 p = v;

		float h = 0.0f;
		float2 n = 0.0f;
				
		float amplitudeSum = 0.0f;

		for (int wi = 0; wi < FragmentWaveCount; ++wi) 
        {
			float2 d = normalize(float2(cos(seed), sin(seed)));

            //float2 d = float2(frac(sin(dot(float2((float) wi,2), float2(12.9898, 78.233)))) * 2 - 1, frac(sin(dot(float2((float) wi * 2,2), float2(12.9898, 78.233)))) * 2 - 1);
            //d = normalize(lerp(normalize(d), normalize(Direction.xy), RandomDirection));

            float scale = (dot(d,Direction.xy) + 1) * 0.5;//0-1 -> 
            scale = lerp(1, scale * ScaleMult, FlowmapMask);

			float x = dot(d, p.xy) * f + Time * speed;
			float wave = a * (exp(FragmentMaxPeak * sin(x) * scale - FragmentPeakOffset) - 1.535);
            //float wave = a * FragmentMaxPeak * sin(x) - FragmentPeakOffset;
			float2 dw = f * d * (FragmentMaxPeak * wave * cos(x));
			
			h += wave;
			p.xy += -dw * a * FragmentDrag;
					
			n += dw;
					
			amplitudeSum += a;
			f *= FragmentFrequencyMult;
			a *= FragmentAmplitudeMult;
			speed *= FragmentSpeedRamp;
			seed += FragmentSeedIter;
		}
				
		float3 output = float3(h, n.x, n.y) / amplitudeSum;
		output.x *= FragmentHeight;

		return output;
    }
};

Functions func;

normal = float3(0.0f, 0.0f, 1.0f);
float height = 0.0f;
float3 fbm = func.fragmentFBM(InWorldPosition);

height = fbm.x;
normal.xy = -fbm.yz;
normal = normalize(normal);

return float3(0.0, 0.0, height);
```

## 带flowmap并且流速不一致
```HLSL
struct Functions{
    float3 fragmentFBM(float3 v) 
    {
		float f = FragmentFrequency;
		float a = FragmentAmplitude;
		float speed = FragmentInitialSpeed;
        float speed2 = FragmentInitialSpeed2;
		float seed = FragmentSeed;
		float3 p = v;

		float h = 0.0f;
		float2 n = 0.0f;
				
		float amplitudeSum = 0.0f;

		for (int wi = 0; wi < FragmentWaveCount; ++wi) 
        {
			float2 d = normalize(float2(cos(seed), sin(seed)));

            //float2 d = float2(frac(sin(dot(float2((float) wi,2), float2(12.9898, 78.233)))) * 2 - 1, frac(sin(dot(float2((float) wi * 2,2), float2(12.9898, 78.233)))) * 2 - 1);
            //d = normalize(lerp(normalize(d), normalize(Direction.xy), RandomDirection));

            float scale = (dot(d,Direction.xy) + 1) * 0.5;//0-1 -> 
            scale = lerp(1, scale * ScaleMult, FlowmapMask);

			float x = dot(d, p.xy) * f + Time * speed;
            float x2 = dot(d, p.xy) * f + Time * speed2;
			float wave = a * (exp(FragmentMaxPeak * sin(x) * scale - FragmentPeakOffset) - 1.535);
            float wave2 = a * (exp(FragmentMaxPeak * sin(x2) * scale - FragmentPeakOffset) - 1.535);
            //float wave = a * FragmentMaxPeak * sin(x) - FragmentPeakOffset;
			float2 dw = f * d * (FragmentMaxPeak * wave * cos(x));
            float2 dw2 = f * d * (FragmentMaxPeak * wave2 * cos(x2));
			
			//h += wave;
            h += lerp(wave, wave2, FlowmapMask);
			//p.xy += -dw * a * FragmentDrag;
            p.xy += -lerp(dw, dw2, FlowmapMask) * a * FragmentDrag;

			//n += dw;		
			n += lerp(dw, dw2, FlowmapMask);
					
			amplitudeSum += a;
			f *= FragmentFrequencyMult;
			a *= FragmentAmplitudeMult;
			speed *= FragmentSpeedRamp;
			seed += FragmentSeedIter;
		}
				
		float3 output = float3(h, n.x, n.y) / amplitudeSum;
		output.x *= FragmentHeight;

		return output;
    }
};

Functions func;

normal = float3(0.0f, 0.0f, 1.0f);
float height = 0.0f;
float3 fbm = func.fragmentFBM(InWorldPosition);

height = fbm.x;
normal.xy = -fbm.yz;
normal = normalize(normal);

return float3(0.0, 0.0, height);
```

![[Pasted image 20250114211753.png]]