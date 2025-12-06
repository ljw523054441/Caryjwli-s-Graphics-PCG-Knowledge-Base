# 原理
![[Pasted image 20250320163137.png]]

![[Pasted image 20250320163113.png]]
# 参考链接
[八面体映射](https://zhuanlan.zhihu.com/p/655725271)
## uv to dir
### Octahedral map
```c++
// For the two equal area functions below to correctly map between eachother
// There needs to be a sign function that returns 1.0 for 0.0 and
// -1.0 for -0.0. This is due to the discontinuity in the octahedral map
// between the upper left triangle and bottom left triangle for example
// (same on right). Check image in RT Gems 16.5.4.2, light purple and dark
// purple for example.
struct Functions{
    float signPreserveZero(float v)
    {
        int i = asint(v);
        return (i < 0) ? -1.0 : 1.0;
    }

    float3 octSphereMap(float2 u)
    {
        u = u * 2.f - 1.f;

        // Compute radius r (branchless)
	    float d = 1.f - (abs(u.x) + abs(u.y));
	    float r = 1.f - abs(d);

        // Compute phi in the first quadrant (branchless, except for the
        // division-by-zero test), using sign(u) to map the result to the
        // correct quadrant below
	    float phi = (r == 0.f) ? 0.f : (PI / 4 * ((abs(u.y) - abs(u.x)) / r + 1.f));

	    float f = r * sqrt(2.f - r * r);

        // abs() around f * cos/sin(phi) is necessary because they can return
        // negative 0 due to floating precision
	    float x = signPreserveZero(u.x) * abs(f * cos(phi));
	    float y = signPreserveZero(u.y) * abs(f * sin(phi));
	    float z = signPreserveZero(d) * (1.f - r * r);

	    return float3(x, y, z);
    }
};

Functions func;

return func.octSphereMap(uv);
```
### Concentric Octahedral map
```c++
// For the two equal area functions below to correctly map between eachother
// There needs to be a sign function that returns 1.0 for 0.0 and
// -1.0 for -0.0. This is due to the discontinuity in the octahedral map
// between the upper left triangle and bottom left triangle for example
// (same on right). Check image in RT Gems 16.5.4.2, light purple and dark
// purple for example.
struct Functions{
    float signPreserveZero(float v)
    {
        int i = asint(v);
        return (i < 0) ? -1.0 : 1.0;
    }

    float3 octSphereMap(float2 u)
    {
        u = u * 2.f - 1.f;

        // Compute radius r (branchless)
	    float d = 1.f - (abs(u.x) + abs(u.y));
	    float r = 1.f - abs(d);

        // Compute phi in the first quadrant (branchless, except for the
        // division-by-zero test), using sign(u) to map the result to the
        // correct quadrant below
	    float phi = (r == 0.f) ? 0.f : (PI / 4 * ((abs(u.y) - abs(u.x)) / r + 1.f));

	    float f = r * sqrt(2.f - r * r);

        // abs() around f * cos/sin(phi) is necessary because they can return
        // negative 0 due to floating precision
	    float x = signPreserveZero(u.x) * abs(f * cos(phi));
	    float y = signPreserveZero(u.y) * abs(f * sin(phi));
	    float z = signPreserveZero(d) * (1.f - r * r);

	    return float3(x, y, z);
    }
};

Functions func;

return func.octSphereMap(uv);
```
### UE5实现的Concentric Octahedral map
```c++
// Based on: [Clarberg 2008, "Fast Equal-Area Mapping of the (Hemi)Sphere using SIMD"]
// Fixed sign bit for UV.y == 0 and removed branch before division by using a small epsilon
// https://fileadmin.cs.lth.se/graphics/research/papers/2008/simdmapping/clarberg_simdmapping08_preprint.pdf
struct Functions{
    float3 EquiAreaSphericalMapping(float2 UV)
    {
	    UV = 2 * UV - 1;
	    float D = 1 - (abs(UV.x) + abs(UV.y));
	    float R = 1 - abs(D);
	    // Branch to avoid dividing by 0.
	    // Only happens with (0.5, 0.5), usually occurs in odd number resolutions which use the very central texel
	    float Phi = R == 0 ? 0 : (PI / 4) * ((abs(UV.y) - abs(UV.x)) / R + 1);
	    float F = R * sqrt(2 - R * R);
	    return float3(
		    F * sign(UV.x) * abs(cos(Phi)),
		    F * sign(UV.y) * abs(sin(Phi)),
		    sign(D) * (1 - R * R)
	    );
    }
};

Functions func;

return func.EquiAreaSphericalMapping(uv);
```

## dir to uv
### Octahedral map
```c++
struct Functions{
    float RTXGISignNotZero(float v)
    {
	    return (v >= 0.f) ? 1.f : -1.f;
    }

    /**
     * 2-component version of RTXGISignNotZero.
     */
    float2 RTXGISignNotZero(float2 v)
    {
    	return float2(RTXGISignNotZero(v.x), RTXGISignNotZero(v.y));
    }

    /**
     * Computes the octant coordinates in the normalized [-1, 1] square, for the given a unit direction vector.
     * The opposite of DDGIGetOctahedralDirection().
     * Used by GetDDGIVolumeIrradiance() in Irradiance.hlsl.
     */
    float2 invOctSphereMap2(float3 direction)
    {
	    float l1norm = abs(direction.x) + abs(direction.y) + abs(direction.z);
	    float2 uv = direction.xy * (1.f / l1norm);
	    if (direction.z < 0.f)
	    {
	    	uv = (1.f - abs(uv.yx)) * RTXGISignNotZero(uv.xy);
	    }
	
	    return uv * 0.5 + 0.5;
    }
};

Functions func;
return func.invOctSphereMap2(dir);
```
### Concentric Octahedral map
```c++
// For the two equal area functions below to correctly map between eachother
// There needs to be a sign function that returns 1.0 for 0.0 and
// -1.0 for -0.0. This is due to the discontinuity in the octahedral map
// between the upper left triangle and bottom left triangle for example
// (same on right). Check image in RT Gems 16.5.4.2, light purple and dark
// purple for example.
struct Functions{
    float signPreserveZero(float v)
    {
        int i = asint(v);
        return (i < 0) ? -1.0 : 1.0;
    }

    float2 invOctSphereMap(float3 dir)
    {
	    float r = sqrt(1.f - abs(dir.z));
	    float phi = atan2(abs(dir.y), abs(dir.x));

	    float2 uv;
	    uv.y = r * phi * 2 / PI;
	    uv.x = r - uv.y;

	    if (dir.z < 0.f)
	    {
		    uv = 1.f - uv.yx;
	    }

	    uv.x *= signPreserveZero(dir.x);
	    uv.y *= signPreserveZero(dir.y);

	    return uv * 0.5f + 0.5f;
    }
};

Functions func;

return func.invOctSphereMap(dir);
```

### UE5实现的Concentric Octahedral map
```c++
// Based on: [Clarberg 2008, "Fast Equal-Area Mapping of the (Hemi)Sphere using SIMD"]
// Removed branch before division by using a small epsilon
// https://fileadmin.cs.lth.se/graphics/research/papers/2008/simdmapping/clarberg_simdmapping08_preprint.pdf
struct Functions{
    float2 InverseEquiAreaSphericalMapping(float3 Direction)
    {
	    // Most use cases of this func generate Direction by diffing two positions and thus unnormalized
	    Direction = normalize(Direction);
	
	    float3 AbsDir = abs(Direction);
	    float R = sqrt(1 - AbsDir.z);
	    float Epsilon = 5.42101086243e-20; // 2^-64 (this avoids 0/0 without changing the rest of the mapping)
	    float x = min(AbsDir.x, AbsDir.y) / (max(AbsDir.x, AbsDir.y) + Epsilon);

	    // Coefficients for 6th degree minimax approximation of atan(x)*2/pi, x=[0,1].
	    const float t1 = 0.406758566246788489601959989e-5f;
	    const float t2 = 0.636226545274016134946890922156f;
	    const float t3 = 0.61572017898280213493197203466e-2f;
	    const float t4 = -0.247333733281268944196501420480f;
	    const float t5 = 0.881770664775316294736387951347e-1f;
	    const float t6 = 0.419038818029165735901852432784e-1f;
	    const float t7 = -0.251390972343483509333252996350e-1f;

	    // Polynomial approximation of atan(x)*2/pi
	    float Phi = t6 + t7 * x;
	    Phi = t5 + Phi * x;
	    Phi = t4 + Phi * x;
	    Phi = t3 + Phi * x;
	    Phi = t2 + Phi * x;
	    Phi = t1 + Phi * x;

	    Phi = (AbsDir.x < AbsDir.y) ? 1 - Phi : Phi;
	    float2 UV = float2(R - Phi * R, Phi * R);
	    UV = (Direction.z < 0) ? 1 - UV.yx : UV;
	    UV = asfloat(asuint(UV) ^ (asuint(Direction.xy) & 0x80000000u));
	    return UV * 0.5 + 0.5;
    }
};

Functions func;

return func.InverseEquiAreaSphericalMapping(dir);
```