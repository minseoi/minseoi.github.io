---
layout: post
title:  "Slice Mesh"
featured-img: test
categories: Unreal
---

메시는 삼각형으로 구성되어 있으니, 삼각형을 자르는 것 부터 알아보자.

UE의 __UKismetProceduralMeshLibrary::SliceProceduralMesh__ 의 코드를 가져와서 분석 해 보았다.

삼각형을 만들기 위해 길이3의 Vectex Array와 Index Array가 있을것이며, 이를 자르는 평면이 있을것이다.

함수 시그니처는 아래와 같이 구성하였다.
```cpp
void Slice(FVector PlanePosition, FVector PlaneNormal, bool bCreateOtherHalf, UCustomSliceComponent*& OutOtherHalfProcMesh);
```

```cpp
// Transform plane from world to local space
FTransform ProcCompToWorld = GetComponentToWorld();
FVector LocalPlanePos = ProcCompToWorld.InverseTransformPosition(PlanePosition);
FVector LocalPlaneNormal = ProcCompToWorld.InverseTransformVectorNoScale(PlaneNormal);
LocalPlaneNormal = LocalPlaneNormal.GetSafeNormal(); // Ensure normalized

FPlane SlicePlane(LocalPlanePos, LocalPlaneNormal);

// Set of sections to add to the 'other half' component
TArray<FProcMeshSection> OtherSections;
// Material for each section of other half
TArray<UMaterialInterface*> OtherMaterials;

// Set of new edges created by clipping polys by plane
TArray<FUtilEdge3D> ClipEdges;
```

```cpp
// Build vertex buffer 
for (int32 BaseVertIndex = 0; BaseVertIndex < NumBaseVerts; BaseVertIndex++)
{
  FProcMeshVertex& BaseVert = BaseSection->ProcVertexBuffer[BaseVertIndex];

  // Calc distance from plane
  VertDistance[BaseVertIndex] = SlicePlane.PlaneDot(BaseVert.Position);

  // See if vert is being kept in this section
  if (VertDistance[BaseVertIndex] > 0.f)
  {
    // Copy to sliced v buffer
    int32 SlicedVertIndex = NewSection.ProcVertexBuffer.Add(BaseVert);
    // Update section bounds
    NewSection.SectionLocalBox += BaseVert.Position;
    // Add to map
    BaseToSlicedVertIndex.Add(BaseVertIndex, SlicedVertIndex);
  }
  // Or add to other half if desired
  else if(NewOtherSection != nullptr)
  {
    int32 SlicedVertIndex = NewOtherSection->ProcVertexBuffer.Add(BaseVert);
    NewOtherSection->SectionLocalBox += BaseVert.Position;
    BaseToOtherSlicedVertIndex.Add(BaseVertIndex, SlicedVertIndex);
  }
}
```

