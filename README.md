# ff-example
fault injection examples

## 1. C to LL (Optimization Level 2)

### 1.1. Sum

#### 1.1.1. Sum inner marker

``` c
extern void __marking_faultinject_int(int);

int main(int argc, char *argv[])
{
  int a = 0, b = 0;
  for (int i = 0; i < argc; i++) {
    __marking_faultinject_int(b);
    b += i;
  }
  printf("%d", b);
}
```

``` llvm
define dso_local i32 @main(i32, i8** nocapture readnone) local_unnamed_addr #0 {
  %3 = icmp sgt i32 %0, 0
  br i1 %3, label %7, label %4

; <label>:4:                                      ; preds = %7, %2
  %5 = phi i32 [ 0, %2 ], [ %10, %7 ]
  %6 = tail call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @"??_C@_02DPKJAMEF@?$CFd?$AA@", i64 0, i64 0), i32 %5)
  ret i32 0

; <label>:7:                                      ; preds = %2, %7
  %8 = phi i32 [ %11, %7 ], [ 0, %2 ]
  %9 = phi i32 [ %10, %7 ], [ 0, %2 ]
  tail call void @__marking_faultinject_int(i32 %9) #3
  %10 = add nuw nsw i32 %8, %9
  %11 = add nuw nsw i32 %8, 1
  %12 = icmp eq i32 %11, %0
  br i1 %12, label %4, label %7
}
```

#### 1.1.2. Sum outer marker

``` c
extern void __marking_faultinject_int(int);
int main(int argc, char *argv[])
{
  int a = 0, b = 0;
  __marking_faultinject_int(b);
  for (int i = 0; i < argc; i++) {
    b += i;
  }
  printf("%d", b);
}
```

``` llvm
define dso_local i32 @main(i32, i8** nocapture readnone) local_unnamed_addr #0 {
  tail call void @__marking_faultinject_int(i32 0) #3
  %3 = icmp sgt i32 %0, 0
  br i1 %3, label %4, label %14

; <label>:4:                                      ; preds = %2
  %5 = add i32 %0, -1
  %6 = zext i32 %5 to i33
  %7 = add i32 %0, -2
  %8 = zext i32 %7 to i33
  %9 = mul i33 %6, %8
  %10 = lshr i33 %9, 1
  %11 = trunc i33 %10 to i32
  %12 = add i32 %11, %0
  %13 = add i32 %12, -1
  br label %14

; <label>:14:                                     ; preds = %4, %2
  %15 = phi i32 [ 0, %2 ], [ %13, %4 ]
  %16 = tail call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @"??_C@_02DPKJAMEF@?$CFd?$AA@", i64 0, i64 0), i32 %15)
  ret i32 0
}
```

#### 1.1.3. Sum inner marker volatile

``` c
extern void __marking_faultinject_int(int);
int main(int argc, char *argv[])
{
  volatile int a = 0, b = 0;
  for (int i = 0; i < argc; i++) {
    __marking_faultinject_int(b);
    b += i;
  }
  printf("%d", b);
}
```

``` llvm
define dso_local i32 @main(i32, i8** nocapture readnone) local_unnamed_addr #0 {
  %3 = alloca i32, align 4
  %4 = alloca i32, align 4
  %5 = bitcast i32* %3 to i8*
  call void @llvm.lifetime.start.p0i8(i64 4, i8* nonnull %5)
  store volatile i32 0, i32* %3, align 4, !tbaa !3
  %6 = bitcast i32* %4 to i8*
  call void @llvm.lifetime.start.p0i8(i64 4, i8* nonnull %6)
  store volatile i32 0, i32* %4, align 4, !tbaa !3
  %7 = icmp sgt i32 %0, 0
  %8 = load volatile i32, i32* %4, align 4, !tbaa !3
  br i1 %7, label %12, label %9

; <label>:9:                                      ; preds = %12, %2
  %10 = phi i32 [ %8, %2 ], [ %18, %12 ]
  %11 = tail call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @"??_C@_02DPKJAMEF@?$CFd?$AA@", i64 0, i64 0), i32 %10)
  call void @llvm.lifetime.end.p0i8(i64 4, i8* nonnull %6)
  call void @llvm.lifetime.end.p0i8(i64 4, i8* nonnull %5)
  ret i32 0

; <label>:12:                                     ; preds = %2, %12
  %13 = phi i32 [ %18, %12 ], [ %8, %2 ]
  %14 = phi i32 [ %17, %12 ], [ 0, %2 ]
  tail call void @__marking_faultinject_int(i32 %13) #4
  %15 = load volatile i32, i32* %4, align 4, !tbaa !3
  %16 = add nsw i32 %15, %14
  store volatile i32 %16, i32* %4, align 4, !tbaa !3
  %17 = add nuw nsw i32 %14, 1
  %18 = load volatile i32, i32* %4, align 4, !tbaa !3
  %19 = icmp eq i32 %17, %0
  br i1 %19, label %9, label %12
}
```

#### 1.1.4. Sum outer marker volatile

``` c
extern void __marking_faultinject_int(int);
int main(int argc, char *argv[])
{
  volatile int a = 0, b = 0;
  __marking_faultinject_int(b);
  for (int i = 0; i < argc; i++) {
    b += i;
  }
  printf("%d", b);
}
```

``` llvm
define dso_local i32 @main(i32, i8** nocapture readnone) local_unnamed_addr #0 {
  %3 = alloca i32, align 4
  %4 = alloca i32, align 4
  %5 = bitcast i32* %3 to i8*
  call void @llvm.lifetime.start.p0i8(i64 4, i8* nonnull %5)
  store volatile i32 0, i32* %3, align 4, !tbaa !3
  %6 = bitcast i32* %4 to i8*
  call void @llvm.lifetime.start.p0i8(i64 4, i8* nonnull %6)
  store volatile i32 0, i32* %4, align 4, !tbaa !3
  %7 = load volatile i32, i32* %4, align 4, !tbaa !3
  tail call void @__marking_faultinject_int(i32 %7) #4
  %8 = icmp sgt i32 %0, 0
  %9 = load volatile i32, i32* %4, align 4, !tbaa !3
  br i1 %8, label %10, label %30

; <label>:10:                                     ; preds = %2
  %11 = add i32 %0, -1
  %12 = and i32 %0, 3
  %13 = icmp ult i32 %11, 3
  br i1 %13, label %16, label %14

; <label>:14:                                     ; preds = %10
  %15 = sub i32 %0, %12
  br label %33

; <label>:16:                                     ; preds = %33, %10
  %17 = phi i32 [ undef, %10 ], [ %48, %33 ]
  %18 = phi i32 [ %9, %10 ], [ %48, %33 ]
  %19 = phi i32 [ 0, %10 ], [ %47, %33 ]
  %20 = icmp eq i32 %12, 0
  br i1 %20, label %30, label %21

; <label>:21:                                     ; preds = %16, %21
  %22 = phi i32 [ %27, %21 ], [ %18, %16 ]
  %23 = phi i32 [ %26, %21 ], [ %19, %16 ]
  %24 = phi i32 [ %28, %21 ], [ %12, %16 ]
  %25 = add nsw i32 %22, %23
  store volatile i32 %25, i32* %4, align 4, !tbaa !3
  %26 = add nuw nsw i32 %23, 1
  %27 = load volatile i32, i32* %4, align 4, !tbaa !3
  %28 = add i32 %24, -1
  %29 = icmp eq i32 %28, 0
  br i1 %29, label %30, label %21, !llvm.loop !7

; <label>:30:                                     ; preds = %16, %21, %2
  %31 = phi i32 [ %9, %2 ], [ %17, %16 ], [ %27, %21 ]
  %32 = tail call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @"??_C@_02DPKJAMEF@?$CFd?$AA@", i64 0, i64 0), i32 %31)
  call void @llvm.lifetime.end.p0i8(i64 4, i8* nonnull %6)
  call void @llvm.lifetime.end.p0i8(i64 4, i8* nonnull %5)
  ret i32 0

; <label>:33:                                     ; preds = %33, %14
  %34 = phi i32 [ %9, %14 ], [ %48, %33 ]
  %35 = phi i32 [ 0, %14 ], [ %47, %33 ]
  %36 = phi i32 [ %15, %14 ], [ %49, %33 ]
  %37 = add nsw i32 %34, %35
  store volatile i32 %37, i32* %4, align 4, !tbaa !3
  %38 = or i32 %35, 1
  %39 = load volatile i32, i32* %4, align 4, !tbaa !3
  %40 = add nsw i32 %39, %38
  store volatile i32 %40, i32* %4, align 4, !tbaa !3
  %41 = or i32 %35, 2
  %42 = load volatile i32, i32* %4, align 4, !tbaa !3
  %43 = add nsw i32 %42, %41
  store volatile i32 %43, i32* %4, align 4, !tbaa !3
  %44 = or i32 %35, 3
  %45 = load volatile i32, i32* %4, align 4, !tbaa !3
  %46 = add nsw i32 %45, %44
  store volatile i32 %46, i32* %4, align 4, !tbaa !3
  %47 = add nuw nsw i32 %35, 4
  %48 = load volatile i32, i32* %4, align 4, !tbaa !3
  %49 = add i32 %36, -4
  %50 = icmp eq i32 %49, 0
  br i1 %50, label %16, label %33
}
```

#### 1.1.5. Sum outer pointer marker

``` c
extern void __marking_faultinject_intptr(int*);
int main(int argc, char *argv[])
{
  int a = 0, b = 0;
  __marking_faultinject_intptr(&b);
  for (int i = 0; i < argc; i++) {
    b += i;
  }
  printf("%d", b);
}
```

``` llvm
define dso_local i32 @main(i32, i8** nocapture readnone) local_unnamed_addr #0 {
  %3 = alloca i32, align 4
  %4 = bitcast i32* %3 to i8*
  call void @llvm.lifetime.start.p0i8(i64 4, i8* nonnull %4) #4
  store i32 0, i32* %3, align 4, !tbaa !3
  call void @__marking_faultinject_intptr(i32* nonnull %3) #4
  %5 = icmp sgt i32 %0, 0
  %6 = load i32, i32* %3, align 4, !tbaa !3
  br i1 %5, label %7, label %18

; <label>:7:                                      ; preds = %2
  %8 = add i32 %6, %0
  %9 = add i32 %0, -1
  %10 = zext i32 %9 to i33
  %11 = add i32 %0, -2
  %12 = zext i32 %11 to i33
  %13 = mul i33 %10, %12
  %14 = lshr i33 %13, 1
  %15 = trunc i33 %14 to i32
  %16 = add i32 %8, %15
  %17 = add i32 %16, -1
  store i32 %17, i32* %3, align 4, !tbaa !3
  br label %18

; <label>:18:                                     ; preds = %7, %2
  %19 = phi i32 [ %17, %7 ], [ %6, %2 ]
  %20 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @"??_C@_02DPKJAMEF@?$CFd?$AA@", i64 0, i64 0), i32 %19)
  call void @llvm.lifetime.end.p0i8(i64 4, i8* nonnull %4) #4
  ret i32 0
}
```

### 1.2. Multiple

#### 1.2.1. Multiple outer pointer marker

``` c
extern void __marking_faultinject_intptr(int*);
int main(int argc, char *argv[])
{
  int  b = 1;
  __marking_faultinject_intptr(&b);
  for (int i = 1; i < argc; i++) {
    b *= i;
  }
  printf("%d", b);
}
```

``` llvm
define dso_local i32 @main(i32, i8** nocapture readnone) local_unnamed_addr #0 {
  %3 = alloca i32, align 4
  %4 = bitcast i32* %3 to i8*
  call void @llvm.lifetime.start.p0i8(i64 4, i8* nonnull %4) #4
  store i32 1, i32* %3, align 4, !tbaa !3
  call void @__marking_faultinject_intptr(i32* nonnull %3) #4
  %5 = icmp sgt i32 %0, 1
  %6 = load i32, i32* %3, align 4, !tbaa !3
  br i1 %5, label %7, label %77

; <label>:7:                                      ; preds = %2
  %8 = add i32 %0, -1
  %9 = icmp ult i32 %8, 8
  br i1 %9, label %10, label %13

; <label>:10:                                     ; preds = %65, %7
  %11 = phi i32 [ %6, %7 ], [ %73, %65 ]
  %12 = phi i32 [ 1, %7 ], [ %15, %65 ]
  br label %80

; <label>:13:                                     ; preds = %7
  %14 = and i32 %8, -8
  %15 = or i32 %14, 1
  %16 = insertelement <4 x i32> <i32 undef, i32 1, i32 1, i32 1>, i32 %6, i32 0
  %17 = add i32 %14, -8
  %18 = lshr exact i32 %17, 3
  %19 = add nuw nsw i32 %18, 1
  %20 = and i32 %19, 3
  %21 = icmp ult i32 %17, 24
  br i1 %21, label %47, label %22

; <label>:22:                                     ; preds = %13
  %23 = sub nsw i32 %19, %20
  br label %24

; <label>:24:                                     ; preds = %24, %22
  %25 = phi <4 x i32> [ %16, %22 ], [ %42, %24 ]
  %26 = phi <4 x i32> [ <i32 1, i32 1, i32 1, i32 1>, %22 ], [ %43, %24 ]
  %27 = phi <4 x i32> [ <i32 1, i32 2, i32 3, i32 4>, %22 ], [ %44, %24 ]
  %28 = phi i32 [ %23, %22 ], [ %45, %24 ]
  %29 = add <4 x i32> %27, <i32 4, i32 4, i32 4, i32 4>
  %30 = mul nsw <4 x i32> %25, %27
  %31 = mul nsw <4 x i32> %26, %29
  %32 = add <4 x i32> %27, <i32 8, i32 8, i32 8, i32 8>
  %33 = add <4 x i32> %27, <i32 12, i32 12, i32 12, i32 12>
  %34 = mul nsw <4 x i32> %30, %32
  %35 = mul nsw <4 x i32> %31, %33
  %36 = add <4 x i32> %27, <i32 16, i32 16, i32 16, i32 16>
  %37 = add <4 x i32> %27, <i32 20, i32 20, i32 20, i32 20>
  %38 = mul nsw <4 x i32> %34, %36
  %39 = mul nsw <4 x i32> %35, %37
  %40 = add <4 x i32> %27, <i32 24, i32 24, i32 24, i32 24>
  %41 = add <4 x i32> %27, <i32 28, i32 28, i32 28, i32 28>
  %42 = mul nsw <4 x i32> %38, %40
  %43 = mul nsw <4 x i32> %39, %41
  %44 = add <4 x i32> %27, <i32 32, i32 32, i32 32, i32 32>
  %45 = add i32 %28, -4
  %46 = icmp eq i32 %45, 0
  br i1 %46, label %47, label %24, !llvm.loop !7

; <label>:47:                                     ; preds = %24, %13
  %48 = phi <4 x i32> [ undef, %13 ], [ %42, %24 ]
  %49 = phi <4 x i32> [ undef, %13 ], [ %43, %24 ]
  %50 = phi <4 x i32> [ %16, %13 ], [ %42, %24 ]
  %51 = phi <4 x i32> [ <i32 1, i32 1, i32 1, i32 1>, %13 ], [ %43, %24 ]
  %52 = phi <4 x i32> [ <i32 1, i32 2, i32 3, i32 4>, %13 ], [ %44, %24 ]
  %53 = icmp eq i32 %20, 0
  br i1 %53, label %65, label %54

; <label>:54:                                     ; preds = %47, %54
  %55 = phi <4 x i32> [ %60, %54 ], [ %50, %47 ]
  %56 = phi <4 x i32> [ %61, %54 ], [ %51, %47 ]
  %57 = phi <4 x i32> [ %62, %54 ], [ %52, %47 ]
  %58 = phi i32 [ %63, %54 ], [ %20, %47 ]
  %59 = add <4 x i32> %57, <i32 4, i32 4, i32 4, i32 4>
  %60 = mul nsw <4 x i32> %55, %57
  %61 = mul nsw <4 x i32> %56, %59
  %62 = add <4 x i32> %57, <i32 8, i32 8, i32 8, i32 8>
  %63 = add i32 %58, -1
  %64 = icmp eq i32 %63, 0
  br i1 %64, label %65, label %54, !llvm.loop !9

; <label>:65:                                     ; preds = %54, %47
  %66 = phi <4 x i32> [ %48, %47 ], [ %60, %54 ]
  %67 = phi <4 x i32> [ %49, %47 ], [ %61, %54 ]
  %68 = mul <4 x i32> %67, %66
  %69 = shufflevector <4 x i32> %68, <4 x i32> undef, <4 x i32> <i32 2, i32 3, i32 undef, i32 undef>
  %70 = mul <4 x i32> %68, %69
  %71 = shufflevector <4 x i32> %70, <4 x i32> undef, <4 x i32> <i32 1, i32 undef, i32 undef, i32 undef>
  %72 = mul <4 x i32> %70, %71
  %73 = extractelement <4 x i32> %72, i32 0
  %74 = icmp eq i32 %8, %14
  br i1 %74, label %75, label %10

; <label>:75:                                     ; preds = %80, %65
  %76 = phi i32 [ %73, %65 ], [ %83, %80 ]
  store i32 %76, i32* %3, align 4, !tbaa !3
  br label %77

; <label>:77:                                     ; preds = %75, %2
  %78 = phi i32 [ %76, %75 ], [ %6, %2 ]
  %79 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @"??_C@_02DPKJAMEF@?$CFd?$AA@", i64 0, i64 0), i32 %78)
  call void @llvm.lifetime.end.p0i8(i64 4, i8* nonnull %4) #4
  ret i32 0

; <label>:80:                                     ; preds = %10, %80
  %81 = phi i32 [ %83, %80 ], [ %11, %10 ]
  %82 = phi i32 [ %84, %80 ], [ %12, %10 ]
  %83 = mul nsw i32 %81, %82
  %84 = add nuw nsw i32 %82, 1
  %85 = icmp eq i32 %84, %0
  br i1 %85, label %75, label %80, !llvm.loop !11
}
```

#### 1.2.1. Multiple outer pointer marker volatile

``` c
extern void __marking_faultinject_intptr(int*);
int main(int argc, char *argv[])
{
  volatile int  b = 1;
  __marking_faultinject_intptr(&b);
  for (volatile int i = 1; i < argc; i++) {
    b *= i;
  }
  printf("%d", b);
}
```

``` llvm
define dso_local i32 @main(i32, i8** nocapture readnone) local_unnamed_addr #0 {
  %3 = alloca i32, align 4
  %4 = alloca i32, align 4
  %5 = bitcast i32* %3 to i8*
  call void @llvm.lifetime.start.p0i8(i64 4, i8* nonnull %5) #4
  store volatile i32 1, i32* %3, align 4, !tbaa !3
  call void @__marking_faultinject_intptr(i32* nonnull %3) #4
  %6 = bitcast i32* %4 to i8*
  call void @llvm.lifetime.start.p0i8(i64 4, i8* nonnull %6)
  store volatile i32 1, i32* %4, align 4, !tbaa !3
  %7 = load volatile i32, i32* %4, align 4, !tbaa !3
  %8 = icmp slt i32 %7, %0
  br i1 %8, label %12, label %9

; <label>:9:                                      ; preds = %12, %2
  call void @llvm.lifetime.end.p0i8(i64 4, i8* nonnull %6)
  %10 = load volatile i32, i32* %3, align 4, !tbaa !3
  %11 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([3 x i8], [3 x i8]* @"??_C@_02DPKJAMEF@?$CFd?$AA@", i64 0, i64 0), i32 %10)
  call void @llvm.lifetime.end.p0i8(i64 4, i8* nonnull %5) #4
  ret i32 0

; <label>:12:                                     ; preds = %2, %12
  %13 = load volatile i32, i32* %4, align 4, !tbaa !3
  %14 = load volatile i32, i32* %3, align 4, !tbaa !3
  %15 = mul nsw i32 %14, %13
  store volatile i32 %15, i32* %3, align 4, !tbaa !3
  %16 = load volatile i32, i32* %4, align 4, !tbaa !3
  %17 = add nsw i32 %16, 1
  store volatile i32 %17, i32* %4, align 4, !tbaa !3
  %18 = load volatile i32, i32* %4, align 4, !tbaa !3
  %19 = icmp slt i32 %18, %0
  br i1 %19, label %12, label %9
}
```

### 1.3. Bubble sort

#### 1.3.1. Bubble sort data swap marker

``` c
extern void __marking_faultinject_int(int);

void bubble_sort(int arr[], int n)
{
  int i, j;
  for (i = 0; i < n - 1; i++)
    for (j = 0; j < n - i - 1; j++)
      if (arr[j] > arr[j + 1]) {
        int tmp = arr[j];
        __marking_faultinject_int(tmp);
        arr[j] = arr[j + 1];
        arr[j + 1] = tmp;
      }
}

int main(int argc, char *argv[])
{
  int arr[] = { 7,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,
                1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,
                2,1,0 };
  int n = sizeof(arr) / sizeof(int);
  bubble_sort(arr, n);
  for (int i = 0; i < n; i++)
    printf("%d ", arr[i]);
}
```

``` llvm
define dso_local void @bubble_sort(i32* nocapture, i32) local_unnamed_addr #0 {
  %3 = icmp sgt i32 %1, 1
  br i1 %3, label %4, label %32

; <label>:4:                                      ; preds = %2
  %5 = add i32 %1, -1
  br label %6

; <label>:6:                                      ; preds = %28, %4
  %7 = phi i32 [ %5, %4 ], [ %30, %28 ]
  %8 = phi i32 [ 0, %4 ], [ %29, %28 ]
  %9 = xor i32 %8, -1
  %10 = add i32 %9, %1
  %11 = icmp sgt i32 %10, 0
  br i1 %11, label %12, label %28

; <label>:12:                                     ; preds = %6
  %13 = load i32, i32* %0, align 4, !tbaa !3
  %14 = zext i32 %7 to i64
  br label %15

; <label>:15:                                     ; preds = %25, %12
  %16 = phi i32 [ %13, %12 ], [ %26, %25 ]
  %17 = phi i64 [ 0, %12 ], [ %18, %25 ]
  %18 = add nuw nsw i64 %17, 1
  %19 = getelementptr inbounds i32, i32* %0, i64 %18
  %20 = load i32, i32* %19, align 4, !tbaa !3
  %21 = icmp sgt i32 %16, %20
  br i1 %21, label %22, label %25

; <label>:22:                                     ; preds = %15
  %23 = getelementptr inbounds i32, i32* %0, i64 %17
  tail call void @__marking_faultinject_int(i32 %16) #4
  %24 = load i32, i32* %19, align 4, !tbaa !3
  store i32 %24, i32* %23, align 4, !tbaa !3
  store i32 %16, i32* %19, align 4, !tbaa !3
  br label %25

; <label>:25:                                     ; preds = %15, %22
  %26 = phi i32 [ %20, %15 ], [ %16, %22 ]
  %27 = icmp eq i64 %18, %14
  br i1 %27, label %28, label %15

; <label>:28:                                     ; preds = %25, %6
  %29 = add nuw nsw i32 %8, 1
  %30 = add i32 %7, -1
  %31 = icmp eq i32 %29, %5
  br i1 %31, label %32, label %6

; <label>:32:                                     ; preds = %28, %2
  ret void
}

define dso_local i32 @main(i32, i8** nocapture readnone) local_unnamed_addr #0 {
  %3 = alloca [97 x i32], align 16
  %4 = bitcast [97 x i32]* %3 to i8*
  call void @llvm.lifetime.start.p0i8(i64 388, i8* nonnull %4) #4
  call void @llvm.memcpy.p0i8.p0i8.i64(i8* nonnull align 16 %4, i8* align 16 bitcast ([97 x i32]* @main.arr to i8*), i64 388, i1 false)
  %5 = getelementptr inbounds [97 x i32], [97 x i32]* %3, i64 0, i64 0
  br label %6

; <label>:6:                                      ; preds = %2, %22
  %7 = phi i64 [ %24, %22 ], [ 96, %2 ]
  %8 = phi i32 [ %23, %22 ], [ 0, %2 ]
  %9 = load i32, i32* %5, align 16, !tbaa !3
  br label %10

; <label>:10:                                     ; preds = %19, %6
  %11 = phi i32 [ %9, %6 ], [ %20, %19 ]
  %12 = phi i64 [ 0, %6 ], [ %13, %19 ]
  %13 = add nuw nsw i64 %12, 1
  %14 = getelementptr inbounds [97 x i32], [97 x i32]* %3, i64 0, i64 %13
  %15 = load i32, i32* %14, align 4, !tbaa !3
  %16 = icmp sgt i32 %11, %15
  br i1 %16, label %17, label %19

; <label>:17:                                     ; preds = %10
  %18 = getelementptr inbounds [97 x i32], [97 x i32]* %3, i64 0, i64 %12
  tail call void @__marking_faultinject_int(i32 %11) #4
  store i32 %15, i32* %18, align 4, !tbaa !3
  store i32 %11, i32* %14, align 4, !tbaa !3
  br label %19

; <label>:19:                                     ; preds = %17, %10
  %20 = phi i32 [ %15, %10 ], [ %11, %17 ]
  %21 = icmp eq i64 %13, %7
  br i1 %21, label %22, label %10

; <label>:22:                                     ; preds = %19
  %23 = add nuw nsw i32 %8, 1
  %24 = add nsw i64 %7, -1
  %25 = icmp eq i32 %23, 96
  br i1 %25, label %27, label %6

; <label>:26:                                     ; preds = %27
  call void @llvm.lifetime.end.p0i8(i64 388, i8* nonnull %4) #4
  ret i32 0

; <label>:27:                                     ; preds = %22, %27
  %28 = phi i64 [ %32, %27 ], [ 0, %22 ]
  %29 = getelementptr inbounds [97 x i32], [97 x i32]* %3, i64 0, i64 %28
  %30 = load i32, i32* %29, align 4, !tbaa !3
  %31 = tail call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([4 x i8], [4 x i8]* @"??_C@_03JDANDILB@?$CFd?5?$AA@", i64 0, i64 0), i32 %30)
  %32 = add nuw nsw i64 %28, 1
  %33 = icmp eq i64 %32, 97
  br i1 %33, label %26, label %27
}
```

#### 1.3.2. Bubble sort data marker

``` c
extern void __marking_faultinject_int(int);

void bubble_sort(int arr[], int n)
{
  int i, j;
  for (i = 0; i < n - 1; i++)
    for (j = 0; j < n - i - 1; j++)
      if (arr[j] > arr[j + 1]) {
        int tmp = arr[j];
        __marking_faultinject_int(arr[j]);
        arr[j] = arr[j + 1];
        arr[j + 1] = tmp;
      }
}

int main(int argc, char *argv[])
{
  int arr[] = { 7,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,
                1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,
                2,1,0 };
  int n = sizeof(arr) / sizeof(int);
  bubble_sort(arr, n);
  for (int i = 0; i < n; i++)
    printf("%d ", arr[i]);
}
```

``` llvm
define dso_local void @bubble_sort(i32* nocapture, i32) local_unnamed_addr #0 {
  %3 = icmp sgt i32 %1, 1
  br i1 %3, label %4, label %32

; <label>:4:                                      ; preds = %2
  %5 = add i32 %1, -1
  br label %6

; <label>:6:                                      ; preds = %28, %4
  %7 = phi i32 [ %5, %4 ], [ %30, %28 ]
  %8 = phi i32 [ 0, %4 ], [ %29, %28 ]
  %9 = xor i32 %8, -1
  %10 = add i32 %9, %1
  %11 = icmp sgt i32 %10, 0
  br i1 %11, label %12, label %28

; <label>:12:                                     ; preds = %6
  %13 = load i32, i32* %0, align 4, !tbaa !3
  %14 = zext i32 %7 to i64
  br label %15

; <label>:15:                                     ; preds = %25, %12
  %16 = phi i32 [ %13, %12 ], [ %26, %25 ]
  %17 = phi i64 [ 0, %12 ], [ %18, %25 ]
  %18 = add nuw nsw i64 %17, 1
  %19 = getelementptr inbounds i32, i32* %0, i64 %18
  %20 = load i32, i32* %19, align 4, !tbaa !3
  %21 = icmp sgt i32 %16, %20
  br i1 %21, label %22, label %25

; <label>:22:                                     ; preds = %15
  %23 = getelementptr inbounds i32, i32* %0, i64 %17
  tail call void @__marking_faultinject_int(i32 %16) #4
  %24 = load i32, i32* %19, align 4, !tbaa !3
  store i32 %24, i32* %23, align 4, !tbaa !3
  store i32 %16, i32* %19, align 4, !tbaa !3
  br label %25

; <label>:25:                                     ; preds = %15, %22
  %26 = phi i32 [ %20, %15 ], [ %16, %22 ]
  %27 = icmp eq i64 %18, %14
  br i1 %27, label %28, label %15

; <label>:28:                                     ; preds = %25, %6
  %29 = add nuw nsw i32 %8, 1
  %30 = add i32 %7, -1
  %31 = icmp eq i32 %29, %5
  br i1 %31, label %32, label %6

; <label>:32:                                     ; preds = %28, %2
  ret void
}

; Function Attrs: nounwind uwtable
define dso_local i32 @main(i32, i8** nocapture readnone) local_unnamed_addr #0 {
  %3 = alloca [97 x i32], align 16
  %4 = bitcast [97 x i32]* %3 to i8*
  call void @llvm.lifetime.start.p0i8(i64 388, i8* nonnull %4) #4
  call void @llvm.memcpy.p0i8.p0i8.i64(i8* nonnull align 16 %4, i8* align 16 bitcast ([97 x i32]* @main.arr to i8*), i64 388, i1 false)
  %5 = getelementptr inbounds [97 x i32], [97 x i32]* %3, i64 0, i64 0
  br label %6

; <label>:6:                                      ; preds = %2, %22
  %7 = phi i64 [ %24, %22 ], [ 96, %2 ]
  %8 = phi i32 [ %23, %22 ], [ 0, %2 ]
  %9 = load i32, i32* %5, align 16, !tbaa !3
  br label %10

; <label>:10:                                     ; preds = %19, %6
  %11 = phi i32 [ %9, %6 ], [ %20, %19 ]
  %12 = phi i64 [ 0, %6 ], [ %13, %19 ]
  %13 = add nuw nsw i64 %12, 1
  %14 = getelementptr inbounds [97 x i32], [97 x i32]* %3, i64 0, i64 %13
  %15 = load i32, i32* %14, align 4, !tbaa !3
  %16 = icmp sgt i32 %11, %15
  br i1 %16, label %17, label %19

; <label>:17:                                     ; preds = %10
  %18 = getelementptr inbounds [97 x i32], [97 x i32]* %3, i64 0, i64 %12
  tail call void @__marking_faultinject_int(i32 %11) #4
  store i32 %15, i32* %18, align 4, !tbaa !3
  store i32 %11, i32* %14, align 4, !tbaa !3
  br label %19

; <label>:19:                                     ; preds = %17, %10
  %20 = phi i32 [ %15, %10 ], [ %11, %17 ]
  %21 = icmp eq i64 %13, %7
  br i1 %21, label %22, label %10

; <label>:22:                                     ; preds = %19
  %23 = add nuw nsw i32 %8, 1
  %24 = add nsw i64 %7, -1
  %25 = icmp eq i32 %23, 96
  br i1 %25, label %27, label %6

; <label>:26:                                     ; preds = %27
  call void @llvm.lifetime.end.p0i8(i64 388, i8* nonnull %4) #4
  ret i32 0

; <label>:27:                                     ; preds = %22, %27
  %28 = phi i64 [ %32, %27 ], [ 0, %22 ]
  %29 = getelementptr inbounds [97 x i32], [97 x i32]* %3, i64 0, i64 %28
  %30 = load i32, i32* %29, align 4, !tbaa !3
  %31 = tail call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([4 x i8], [4 x i8]* @"??_C@_03JDANDILB@?$CFd?5?$AA@", i64 0, i64 0), i32 %30)
  %32 = add nuw nsw i64 %28, 1
  %33 = icmp eq i64 %32, 97
  br i1 %33, label %26, label %27
}
```

---

## 2. C to LL (Optimization Level Zero)

### 2.1. Bubble sort

#### 2.1.1. Bubble sort array element marker

``` c
extern void __marking_faultinject_int(int);
void bubble_sort(int arr[], int n)
{
  int i, j;
  for (i = 0; i < n - 1; i++)
    for (j = 0; j < n - i - 1; j++)
      if (arr[j] > arr[j + 1]) {
        int tmp = arr[j];
        __marking_faultinject_int(arr[j]);
        arr[j] = arr[j + 1];
        arr[j + 1] = tmp;
      }
}

int main(int argc, char *argv[])
{
  int arr[] = { 7,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,
                1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,
                2,1,0 };
  int n = sizeof(arr) / sizeof(int);
  bubble_sort(arr, n);
  for (int i = 0; i < n; i++)
    printf("%d ", arr[i]);
}
```

``` llvm
define dso_local void @bubble_sort(i32*, i32) #0 {
  %3 = alloca i32, align 4
  %4 = alloca i32*, align 8
  %5 = alloca i32, align 4
  %6 = alloca i32, align 4
  %7 = alloca i32, align 4
  store i32 %1, i32* %3, align 4
  store i32* %0, i32** %4, align 8
  store i32 0, i32* %5, align 4
  br label %8

; <label>:8:                                      ; preds = %66, %2
  %9 = load i32, i32* %5, align 4
  %10 = load i32, i32* %3, align 4
  %11 = sub nsw i32 %10, 1
  %12 = icmp slt i32 %9, %11
  br i1 %12, label %13, label %69

; <label>:13:                                     ; preds = %8
  store i32 0, i32* %6, align 4
  br label %14

; <label>:14:                                     ; preds = %62, %13
  %15 = load i32, i32* %6, align 4
  %16 = load i32, i32* %3, align 4
  %17 = load i32, i32* %5, align 4
  %18 = sub nsw i32 %16, %17
  %19 = sub nsw i32 %18, 1
  %20 = icmp slt i32 %15, %19
  br i1 %20, label %21, label %65

; <label>:21:                                     ; preds = %14
  %22 = load i32*, i32** %4, align 8
  %23 = load i32, i32* %6, align 4
  %24 = sext i32 %23 to i64
  %25 = getelementptr inbounds i32, i32* %22, i64 %24
  %26 = load i32, i32* %25, align 4
  %27 = load i32*, i32** %4, align 8
  %28 = load i32, i32* %6, align 4
  %29 = add nsw i32 %28, 1
  %30 = sext i32 %29 to i64
  %31 = getelementptr inbounds i32, i32* %27, i64 %30
  %32 = load i32, i32* %31, align 4
  %33 = icmp sgt i32 %26, %32
  br i1 %33, label %34, label %61

; <label>:34:                                     ; preds = %21
  %35 = load i32*, i32** %4, align 8
  %36 = load i32, i32* %6, align 4
  %37 = sext i32 %36 to i64
  %38 = getelementptr inbounds i32, i32* %35, i64 %37
  %39 = load i32, i32* %38, align 4
  store i32 %39, i32* %7, align 4
  %40 = load i32*, i32** %4, align 8
  %41 = load i32, i32* %6, align 4
  %42 = sext i32 %41 to i64
  %43 = getelementptr inbounds i32, i32* %40, i64 %42
  %44 = load i32, i32* %43, align 4
  call void @__marking_faultinject_int(i32 %44)
  %45 = load i32*, i32** %4, align 8
  %46 = load i32, i32* %6, align 4
  %47 = add nsw i32 %46, 1
  %48 = sext i32 %47 to i64
  %49 = getelementptr inbounds i32, i32* %45, i64 %48
  %50 = load i32, i32* %49, align 4
  %51 = load i32*, i32** %4, align 8
  %52 = load i32, i32* %6, align 4
  %53 = sext i32 %52 to i64
  %54 = getelementptr inbounds i32, i32* %51, i64 %53
  store i32 %50, i32* %54, align 4
  %55 = load i32, i32* %7, align 4
  %56 = load i32*, i32** %4, align 8
  %57 = load i32, i32* %6, align 4
  %58 = add nsw i32 %57, 1
  %59 = sext i32 %58 to i64
  %60 = getelementptr inbounds i32, i32* %56, i64 %59
  store i32 %55, i32* %60, align 4
  br label %61

; <label>:61:                                     ; preds = %34, %21
  br label %62

; <label>:62:                                     ; preds = %61
  %63 = load i32, i32* %6, align 4
  %64 = add nsw i32 %63, 1
  store i32 %64, i32* %6, align 4
  br label %14

; <label>:65:                                     ; preds = %14
  br label %66

; <label>:66:                                     ; preds = %65
  %67 = load i32, i32* %5, align 4
  %68 = add nsw i32 %67, 1
  store i32 %68, i32* %5, align 4
  br label %8

; <label>:69:                                     ; preds = %8
  ret void
}
```

#### 2.1.2. Bubble sort array element pointer marker

``` c
void bubble_sort(int arr[], int n)
{
  int i, j;
  for (i = 0; i < n - 1; i++)
    for (j = 0; j < n - i - 1; j++)
      if (arr[j] > arr[j + 1]) {
        int tmp = arr[j];
        __marking_faultinject_intptr(&arr[j]);
        arr[j] = arr[j + 1];
        arr[j + 1] = tmp;
      }
}

int main(int argc, char *argv[])
{
  int arr[] = { 7,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,
                1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,
                2,1,0 };
  int n = sizeof(arr) / sizeof(int);
  bubble_sort(arr, n);
  for (int i = 0; i < n; i++)
    printf("%d ", arr[i]);
}
```

``` llvm
define dso_local void @bubble_sort(i32*, i32) #0 {
  %3 = alloca i32, align 4
  %4 = alloca i32*, align 8
  %5 = alloca i32, align 4
  %6 = alloca i32, align 4
  %7 = alloca i32, align 4
  store i32 %1, i32* %3, align 4
  store i32* %0, i32** %4, align 8
  store i32 0, i32* %5, align 4
  br label %8

; <label>:8:                                      ; preds = %65, %2
  %9 = load i32, i32* %5, align 4
  %10 = load i32, i32* %3, align 4
  %11 = sub nsw i32 %10, 1
  %12 = icmp slt i32 %9, %11
  br i1 %12, label %13, label %68

; <label>:13:                                     ; preds = %8
  store i32 0, i32* %6, align 4
  br label %14

; <label>:14:                                     ; preds = %61, %13
  %15 = load i32, i32* %6, align 4
  %16 = load i32, i32* %3, align 4
  %17 = load i32, i32* %5, align 4
  %18 = sub nsw i32 %16, %17
  %19 = sub nsw i32 %18, 1
  %20 = icmp slt i32 %15, %19
  br i1 %20, label %21, label %64

; <label>:21:                                     ; preds = %14
  %22 = load i32*, i32** %4, align 8
  %23 = load i32, i32* %6, align 4
  %24 = sext i32 %23 to i64
  %25 = getelementptr inbounds i32, i32* %22, i64 %24
  %26 = load i32, i32* %25, align 4
  %27 = load i32*, i32** %4, align 8
  %28 = load i32, i32* %6, align 4
  %29 = add nsw i32 %28, 1
  %30 = sext i32 %29 to i64
  %31 = getelementptr inbounds i32, i32* %27, i64 %30
  %32 = load i32, i32* %31, align 4
  %33 = icmp sgt i32 %26, %32
  br i1 %33, label %34, label %60

; <label>:34:                                     ; preds = %21
  %35 = load i32*, i32** %4, align 8
  %36 = load i32, i32* %6, align 4
  %37 = sext i32 %36 to i64
  %38 = getelementptr inbounds i32, i32* %35, i64 %37
  %39 = load i32, i32* %38, align 4
  store i32 %39, i32* %7, align 4
  %40 = load i32*, i32** %4, align 8
  %41 = load i32, i32* %6, align 4
  %42 = sext i32 %41 to i64
  %43 = getelementptr inbounds i32, i32* %40, i64 %42
  call void @__marking_faultinject_intptr(i32* %43)
  %44 = load i32*, i32** %4, align 8
  %45 = load i32, i32* %6, align 4
  %46 = add nsw i32 %45, 1
  %47 = sext i32 %46 to i64
  %48 = getelementptr inbounds i32, i32* %44, i64 %47
  %49 = load i32, i32* %48, align 4
  %50 = load i32*, i32** %4, align 8
  %51 = load i32, i32* %6, align 4
  %52 = sext i32 %51 to i64
  %53 = getelementptr inbounds i32, i32* %50, i64 %52
  store i32 %49, i32* %53, align 4
  %54 = load i32, i32* %7, align 4
  %55 = load i32*, i32** %4, align 8
  %56 = load i32, i32* %6, align 4
  %57 = add nsw i32 %56, 1
  %58 = sext i32 %57 to i64
  %59 = getelementptr inbounds i32, i32* %55, i64 %58
  store i32 %54, i32* %59, align 4
  br label %60

; <label>:60:                                     ; preds = %34, %21
  br label %61

; <label>:61:                                     ; preds = %60
  %62 = load i32, i32* %6, align 4
  %63 = add nsw i32 %62, 1
  store i32 %63, i32* %6, align 4
  br label %14

; <label>:64:                                     ; preds = %14
  br label %65

; <label>:65:                                     ; preds = %64
  %66 = load i32, i32* %5, align 4
  %67 = add nsw i32 %66, 1
  store i32 %67, i32* %5, align 4
  br label %8

; <label>:68:                                     ; preds = %8
  ret void
}
```

#### Differ 2.1.1 vs 2.1.2

``` llvm
; 2.1.1
  %43 = getelementptr inbounds i32, i32* %40, i64 %42
  %44 = load i32, i32* %43, align 4
  call void @__marking_faultinject_int(i32 %44)
```

``` llvm
; 2.1.2
  %43 = getelementptr inbounds i32, i32* %40, i64 %42
  call void @__marking_faultinject_intptr(i32* %43)
```

#### 2.1.3. Bubble sort swap temporary-variable marker

``` c
void bubble_sort(int arr[], int n)
{
  int i, j;
  for (i = 0; i < n - 1; i++)
    for (j = 0; j < n - i - 1; j++)
      if (arr[j] > arr[j + 1]) {
        int tmp = arr[j];
        __marking_faultinject_int(tmp);
        arr[j] = arr[j + 1];
        arr[j + 1] = tmp;
      }
}

int main(int argc, char *argv[])
{
  int arr[] = { 7,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,
                1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,
                2,1,0 };
  int n = sizeof(arr) / sizeof(int);
  bubble_sort(arr, n);
  for (int i = 0; i < n; i++)
    printf("%d ", arr[i]);
}
```

``` llvm
define dso_local void @bubble_sort(i32*, i32) #0 {
  %3 = alloca i32, align 4
  %4 = alloca i32*, align 8
  %5 = alloca i32, align 4
  %6 = alloca i32, align 4
  %7 = alloca i32, align 4
  store i32 %1, i32* %3, align 4
  store i32* %0, i32** %4, align 8
  store i32 0, i32* %5, align 4
  br label %8

; <label>:8:                                      ; preds = %62, %2
  %9 = load i32, i32* %5, align 4
  %10 = load i32, i32* %3, align 4
  %11 = sub nsw i32 %10, 1
  %12 = icmp slt i32 %9, %11
  br i1 %12, label %13, label %65

; <label>:13:                                     ; preds = %8
  store i32 0, i32* %6, align 4
  br label %14

; <label>:14:                                     ; preds = %58, %13
  %15 = load i32, i32* %6, align 4
  %16 = load i32, i32* %3, align 4
  %17 = load i32, i32* %5, align 4
  %18 = sub nsw i32 %16, %17
  %19 = sub nsw i32 %18, 1
  %20 = icmp slt i32 %15, %19
  br i1 %20, label %21, label %61

; <label>:21:                                     ; preds = %14
  %22 = load i32*, i32** %4, align 8
  %23 = load i32, i32* %6, align 4
  %24 = sext i32 %23 to i64
  %25 = getelementptr inbounds i32, i32* %22, i64 %24
  %26 = load i32, i32* %25, align 4
  %27 = load i32*, i32** %4, align 8
  %28 = load i32, i32* %6, align 4
  %29 = add nsw i32 %28, 1
  %30 = sext i32 %29 to i64
  %31 = getelementptr inbounds i32, i32* %27, i64 %30
  %32 = load i32, i32* %31, align 4
  %33 = icmp sgt i32 %26, %32
  br i1 %33, label %34, label %57

; <label>:34:                                     ; preds = %21
  %35 = load i32*, i32** %4, align 8
  %36 = load i32, i32* %6, align 4
  %37 = sext i32 %36 to i64
  %38 = getelementptr inbounds i32, i32* %35, i64 %37
  %39 = load i32, i32* %38, align 4
  store i32 %39, i32* %7, align 4
  %40 = load i32, i32* %7, align 4
  call void @__marking_faultinject_int(i32 %40)
  %41 = load i32*, i32** %4, align 8
  %42 = load i32, i32* %6, align 4
  %43 = add nsw i32 %42, 1
  %44 = sext i32 %43 to i64
  %45 = getelementptr inbounds i32, i32* %41, i64 %44
  %46 = load i32, i32* %45, align 4
  %47 = load i32*, i32** %4, align 8
  %48 = load i32, i32* %6, align 4
  %49 = sext i32 %48 to i64
  %50 = getelementptr inbounds i32, i32* %47, i64 %49
  store i32 %46, i32* %50, align 4
  %51 = load i32, i32* %7, align 4
  %52 = load i32*, i32** %4, align 8
  %53 = load i32, i32* %6, align 4
  %54 = add nsw i32 %53, 1
  %55 = sext i32 %54 to i64
  %56 = getelementptr inbounds i32, i32* %52, i64 %55
  store i32 %51, i32* %56, align 4
  br label %57

; <label>:57:                                     ; preds = %34, %21
  br label %58

; <label>:58:                                     ; preds = %57
  %59 = load i32, i32* %6, align 4
  %60 = add nsw i32 %59, 1
  store i32 %60, i32* %6, align 4
  br label %14

; <label>:61:                                     ; preds = %14
  br label %62

; <label>:62:                                     ; preds = %61
  %63 = load i32, i32* %5, align 4
  %64 = add nsw i32 %63, 1
  store i32 %64, i32* %5, align 4
  br label %8

; <label>:65:                                     ; preds = %8
  ret void
}
```

#### 2.1.4. Bubble sort swap temporary-variable pointer

``` c
void bubble_sort(int arr[], int n)
{
  int i, j;
  for (i = 0; i < n - 1; i++)
    for (j = 0; j < n - i - 1; j++)
      if (arr[j] > arr[j + 1]) {
        int tmp = arr[j];
        __marking_faultinject_intptr(&tmp);
        arr[j] = arr[j + 1];
        arr[j + 1] = tmp;
      }
}

int main(int argc, char *argv[])
{
  int arr[] = { 7,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,
                1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,
                2,1,0 };
  int n = sizeof(arr) / sizeof(int);
  bubble_sort(arr, n);
  for (int i = 0; i < n; i++)
    printf("%d ", arr[i]);
}
```

``` llvm
define dso_local void @bubble_sort(i32*, i32) #0 {
  %3 = alloca i32, align 4
  %4 = alloca i32*, align 8
  %5 = alloca i32, align 4
  %6 = alloca i32, align 4
  %7 = alloca i32, align 4
  store i32 %1, i32* %3, align 4
  store i32* %0, i32** %4, align 8
  store i32 0, i32* %5, align 4
  br label %8

; <label>:8:                                      ; preds = %61, %2
  %9 = load i32, i32* %5, align 4
  %10 = load i32, i32* %3, align 4
  %11 = sub nsw i32 %10, 1
  %12 = icmp slt i32 %9, %11
  br i1 %12, label %13, label %64

; <label>:13:                                     ; preds = %8
  store i32 0, i32* %6, align 4
  br label %14

; <label>:14:                                     ; preds = %57, %13
  %15 = load i32, i32* %6, align 4
  %16 = load i32, i32* %3, align 4
  %17 = load i32, i32* %5, align 4
  %18 = sub nsw i32 %16, %17
  %19 = sub nsw i32 %18, 1
  %20 = icmp slt i32 %15, %19
  br i1 %20, label %21, label %60

; <label>:21:                                     ; preds = %14
  %22 = load i32*, i32** %4, align 8
  %23 = load i32, i32* %6, align 4
  %24 = sext i32 %23 to i64
  %25 = getelementptr inbounds i32, i32* %22, i64 %24
  %26 = load i32, i32* %25, align 4
  %27 = load i32*, i32** %4, align 8
  %28 = load i32, i32* %6, align 4
  %29 = add nsw i32 %28, 1
  %30 = sext i32 %29 to i64
  %31 = getelementptr inbounds i32, i32* %27, i64 %30
  %32 = load i32, i32* %31, align 4
  %33 = icmp sgt i32 %26, %32
  br i1 %33, label %34, label %56

; <label>:34:                                     ; preds = %21
  %35 = load i32*, i32** %4, align 8
  %36 = load i32, i32* %6, align 4
  %37 = sext i32 %36 to i64
  %38 = getelementptr inbounds i32, i32* %35, i64 %37
  %39 = load i32, i32* %38, align 4
  store i32 %39, i32* %7, align 4
  call void @__marking_faultinject_intptr(i32* %7)
  %40 = load i32*, i32** %4, align 8
  %41 = load i32, i32* %6, align 4
  %42 = add nsw i32 %41, 1
  %43 = sext i32 %42 to i64
  %44 = getelementptr inbounds i32, i32* %40, i64 %43
  %45 = load i32, i32* %44, align 4
  %46 = load i32*, i32** %4, align 8
  %47 = load i32, i32* %6, align 4
  %48 = sext i32 %47 to i64
  %49 = getelementptr inbounds i32, i32* %46, i64 %48
  store i32 %45, i32* %49, align 4
  %50 = load i32, i32* %7, align 4
  %51 = load i32*, i32** %4, align 8
  %52 = load i32, i32* %6, align 4
  %53 = add nsw i32 %52, 1
  %54 = sext i32 %53 to i64
  %55 = getelementptr inbounds i32, i32* %51, i64 %54
  store i32 %50, i32* %55, align 4
  br label %56

; <label>:56:                                     ; preds = %34, %21
  br label %57

; <label>:57:                                     ; preds = %56
  %58 = load i32, i32* %6, align 4
  %59 = add nsw i32 %58, 1
  store i32 %59, i32* %6, align 4
  br label %14

; <label>:60:                                     ; preds = %14
  br label %61

; <label>:61:                                     ; preds = %60
  %62 = load i32, i32* %5, align 4
  %63 = add nsw i32 %62, 1
  store i32 %63, i32* %5, align 4
  br label %8

; <label>:64:                                     ; preds = %8
  ret void
}
```

#### Differ 2.1.3 vs 2.1.4

#### 2.1.5 Declare variable entry point with pointer marker

``` c
extern void __marking_faultinject_intptr(int*);

void bubble_sort(int arr[], int n)
{
  int i, j, tmp;
  __marking_faultinject_intptr(&tmp);
  for (i = 0; i < n - 1; i++)
    for (j = 0; j < n - i - 1; j++)
      if (arr[j] > arr[j + 1]) {
        tmp = arr[j];
        arr[j] = arr[j + 1];
        arr[j + 1] = tmp;
      }
}

int main(int argc, char *argv[])
{
  int arr[] = { 7,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,
                1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,
                2,1,0 };
  int n = sizeof(arr) / sizeof(int);
  bubble_sort(arr, n);
  for (int i = 0; i < n; i++)
    printf("%d ", arr[i]);
}
```

``` llvm
define dso_local void @bubble_sort(i32*, i32) #0 {
  %3 = alloca i32, align 4
  %4 = alloca i32*, align 8
  %5 = alloca i32, align 4
  %6 = alloca i32, align 4
  %7 = alloca i32, align 4
  store i32 %1, i32* %3, align 4
  store i32* %0, i32** %4, align 8
  call void @__marking_faultinject_intptr(i32* %7)
  store i32 0, i32* %5, align 4
  br label %8

; <label>:8:                                      ; preds = %61, %2
  %9 = load i32, i32* %5, align 4
  %10 = load i32, i32* %3, align 4
  %11 = sub nsw i32 %10, 1
  %12 = icmp slt i32 %9, %11
  br i1 %12, label %13, label %64

; <label>:13:                                     ; preds = %8
  store i32 0, i32* %6, align 4
  br label %14

; <label>:14:                                     ; preds = %57, %13
  %15 = load i32, i32* %6, align 4
  %16 = load i32, i32* %3, align 4
  %17 = load i32, i32* %5, align 4
  %18 = sub nsw i32 %16, %17
  %19 = sub nsw i32 %18, 1
  %20 = icmp slt i32 %15, %19
  br i1 %20, label %21, label %60

; <label>:21:                                     ; preds = %14
  %22 = load i32*, i32** %4, align 8
  %23 = load i32, i32* %6, align 4
  %24 = sext i32 %23 to i64
  %25 = getelementptr inbounds i32, i32* %22, i64 %24
  %26 = load i32, i32* %25, align 4
  %27 = load i32*, i32** %4, align 8
  %28 = load i32, i32* %6, align 4
  %29 = add nsw i32 %28, 1
  %30 = sext i32 %29 to i64
  %31 = getelementptr inbounds i32, i32* %27, i64 %30
  %32 = load i32, i32* %31, align 4
  %33 = icmp sgt i32 %26, %32
  br i1 %33, label %34, label %56

; <label>:34:                                     ; preds = %21
  %35 = load i32*, i32** %4, align 8
  %36 = load i32, i32* %6, align 4
  %37 = sext i32 %36 to i64
  %38 = getelementptr inbounds i32, i32* %35, i64 %37
  %39 = load i32, i32* %38, align 4
  store i32 %39, i32* %7, align 4                         ; <<===========
  %40 = load i32*, i32** %4, align 8
  %41 = load i32, i32* %6, align 4
  %42 = add nsw i32 %41, 1
  %43 = sext i32 %42 to i64
  %44 = getelementptr inbounds i32, i32* %40, i64 %43
  %45 = load i32, i32* %44, align 4
  %46 = load i32*, i32** %4, align 8
  %47 = load i32, i32* %6, align 4
  %48 = sext i32 %47 to i64
  %49 = getelementptr inbounds i32, i32* %46, i64 %48
  store i32 %45, i32* %49, align 4
  %50 = load i32, i32* %7, align 4                        ; <<===========
  %51 = load i32*, i32** %4, align 8
  %52 = load i32, i32* %6, align 4
  %53 = add nsw i32 %52, 1
  %54 = sext i32 %53 to i64
  %55 = getelementptr inbounds i32, i32* %51, i64 %54
  store i32 %50, i32* %55, align 4
  br label %56

; <label>:56:                                     ; preds = %34, %21
  br label %57

; <label>:57:                                     ; preds = %56
  %58 = load i32, i32* %6, align 4
  %59 = add nsw i32 %58, 1
  store i32 %59, i32* %6, align 4
  br label %14

; <label>:60:                                     ; preds = %14
  br label %61

; <label>:61:                                     ; preds = %60
  %62 = load i32, i32* %5, align 4
  %63 = add nsw i32 %62, 1
  store i32 %63, i32* %5, align 4
  br label %8

; <label>:64:                                     ; preds = %8
  ret void
}
```

#### 2.1.6. Iterator marker

``` c
extern void __marking_faultinject_intptr(int*);

void bubble_sort(int arr[], int n)
{
  int i, j, tmp;
  __marking_faultinject_intptr(&j);
  for (i = 0; i < n - 1; i++)
    for (j = 0; j < n - i - 1; j++)
      if (arr[j] > arr[j + 1]) {
        tmp = arr[j];
        arr[j] = arr[j + 1];
        arr[j + 1] = tmp;
      }
}

int main(int argc, char *argv[])
{
  int arr[] = { 7,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,
                1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,2,1,0,8,0,5,4,1,2,4,3,10,99,55,10,0,1,9,8,7,6,5,4,3,
                2,1,0 };
  int n = sizeof(arr) / sizeof(int);
  bubble_sort(arr, n);
  for (int i = 0; i < n; i++)
    printf("%d ", arr[i]);
}
```

``` llvm
define dso_local void @bubble_sort(i32*, i32) #0 {
  %3 = alloca i32, align 4
  %4 = alloca i32*, align 8
  %5 = alloca i32, align 4
  %6 = alloca i32, align 4
  %7 = alloca i32, align 4
  store i32 %1, i32* %3, align 4
  store i32* %0, i32** %4, align 8
  call void @__marking_faultinject_intptr(i32* %6)
  store i32 0, i32* %5, align 4
  br label %8

; <label>:8:                                      ; preds = %61, %2
  %9 = load i32, i32* %5, align 4
  %10 = load i32, i32* %3, align 4
  %11 = sub nsw i32 %10, 1
  %12 = icmp slt i32 %9, %11
  br i1 %12, label %13, label %64

; <label>:13:                                     ; preds = %8
  store i32 0, i32* %6, align 4
  br label %14

; <label>:14:                                     ; preds = %57, %13
  %15 = load i32, i32* %6, align 4
  %16 = load i32, i32* %3, align 4
  %17 = load i32, i32* %5, align 4
  %18 = sub nsw i32 %16, %17
  %19 = sub nsw i32 %18, 1
  %20 = icmp slt i32 %15, %19
  br i1 %20, label %21, label %60

; <label>:21:                                     ; preds = %14
  %22 = load i32*, i32** %4, align 8
  %23 = load i32, i32* %6, align 4
  %24 = sext i32 %23 to i64
  %25 = getelementptr inbounds i32, i32* %22, i64 %24
  %26 = load i32, i32* %25, align 4
  %27 = load i32*, i32** %4, align 8
  %28 = load i32, i32* %6, align 4
  %29 = add nsw i32 %28, 1
  %30 = sext i32 %29 to i64
  %31 = getelementptr inbounds i32, i32* %27, i64 %30
  %32 = load i32, i32* %31, align 4
  %33 = icmp sgt i32 %26, %32
  br i1 %33, label %34, label %56

; <label>:34:                                     ; preds = %21
  %35 = load i32*, i32** %4, align 8
  %36 = load i32, i32* %6, align 4
  %37 = sext i32 %36 to i64
  %38 = getelementptr inbounds i32, i32* %35, i64 %37
  %39 = load i32, i32* %38, align 4
  store i32 %39, i32* %7, align 4
  %40 = load i32*, i32** %4, align 8
  %41 = load i32, i32* %6, align 4
  %42 = add nsw i32 %41, 1
  %43 = sext i32 %42 to i64
  %44 = getelementptr inbounds i32, i32* %40, i64 %43
  %45 = load i32, i32* %44, align 4
  %46 = load i32*, i32** %4, align 8
  %47 = load i32, i32* %6, align 4
  %48 = sext i32 %47 to i64
  %49 = getelementptr inbounds i32, i32* %46, i64 %48
  store i32 %45, i32* %49, align 4
  %50 = load i32, i32* %7, align 4
  %51 = load i32*, i32** %4, align 8
  %52 = load i32, i32* %6, align 4
  %53 = add nsw i32 %52, 1
  %54 = sext i32 %53 to i64
  %55 = getelementptr inbounds i32, i32* %51, i64 %54
  store i32 %50, i32* %55, align 4
  br label %56

; <label>:56:                                     ; preds = %34, %21
  br label %57

; <label>:57:                                     ; preds = %56
  %58 = load i32, i32* %6, align 4
  %59 = add nsw i32 %58, 1
  store i32 %59, i32* %6, align 4
  br label %14

; <label>:60:                                     ; preds = %14
  br label %61

; <label>:61:                                     ; preds = %60
  %62 = load i32, i32* %5, align 4
  %63 = add nsw i32 %62, 1
  store i32 %63, i32* %5, align 4
  br label %8

; <label>:64:                                     ; preds = %8
  ret void
}
```

``` llvm
SELECT :   %58 = load i32, i32* %6, align 4
SELECT :   %52 = load i32, i32* %6, align 4
SELECT :   %47 = load i32, i32* %6, align 4
SELECT :   %41 = load i32, i32* %6, align 4
SELECT :   %36 = load i32, i32* %6, align 4
SELECT :   %28 = load i32, i32* %6, align 4
SELECT :   %23 = load i32, i32* %6, align 4
SELECT :   %15 = load i32, i32* %6, align 4

; Function Attrs: noinline nounwind optnone uwtable
define dso_local void @bubble_sort(i32*, i32) #0 {
  %xor_marker = alloca i32
  store i32 16, i32* %xor_marker
  %3 = alloca i32, align 4
  %4 = alloca i32*, align 8
  %5 = alloca i32, align 4
  %6 = alloca i32, align 4
  %7 = alloca i32, align 4
  store i32 %1, i32* %3, align 4
  store i32* %0, i32** %4, align 8
  call void @__marking_faultinject_intptr(i32* %6)
  store i32 0, i32* %5, align 4
  br label %8

; <label>:8:                                      ; preds = %61, %2
  %9 = load i32, i32* %5, align 4
  %10 = load i32, i32* %3, align 4
  %11 = sub nsw i32 %10, 1
  %12 = icmp slt i32 %9, %11
  br i1 %12, label %13, label %64

; <label>:13:                                     ; preds = %8
  store i32 0, i32* %6, align 4
  br label %14

; <label>:14:                                     ; preds = %57, %13
  %15 = load i32, i32* %6, align 4
  %16 = load i32, i32* %3, align 4
  %17 = load i32, i32* %5, align 4
  %18 = sub nsw i32 %16, %17
  %19 = sub nsw i32 %18, 1
  %20 = icmp slt i32 %15, %19
  br i1 %20, label %21, label %60

; <label>:21:                                     ; preds = %14
  %22 = load i32*, i32** %4, align 8
  %23 = load i32, i32* %6, align 4
  %24 = sext i32 %23 to i64
  %25 = getelementptr inbounds i32, i32* %22, i64 %24
  %26 = load i32, i32* %25, align 4
  %27 = load i32*, i32** %4, align 8
  %28 = load i32, i32* %6, align 4                ; <<=====
  %xor_val = load i32, i32* %xor_marker           ;
  %rfi = xor i32 %28, %xor_val                    ;
  store i32 0, i32* %xor_marker                   ; =====>>
  %29 = add nsw i32 %rfi, 1                       ; <<===>>
  %30 = sext i32 %29 to i64
  %31 = getelementptr inbounds i32, i32* %27, i64 %30
  %32 = load i32, i32* %31, align 4
  %33 = icmp sgt i32 %26, %32
  br i1 %33, label %34, label %56

; <label>:34:                                     ; preds = %21
  %35 = load i32*, i32** %4, align 8
  %36 = load i32, i32* %6, align 4
  %37 = sext i32 %36 to i64
  %38 = getelementptr inbounds i32, i32* %35, i64 %37
  %39 = load i32, i32* %38, align 4
  store i32 %39, i32* %7, align 4
  %40 = load i32*, i32** %4, align 8
  %41 = load i32, i32* %6, align 4
  %42 = add nsw i32 %41, 1
  %43 = sext i32 %42 to i64
  %44 = getelementptr inbounds i32, i32* %40, i64 %43
  %45 = load i32, i32* %44, align 4
  %46 = load i32*, i32** %4, align 8
  %47 = load i32, i32* %6, align 4
  %48 = sext i32 %47 to i64
  %49 = getelementptr inbounds i32, i32* %46, i64 %48
  store i32 %45, i32* %49, align 4
  %50 = load i32, i32* %7, align 4
  %51 = load i32*, i32** %4, align 8
  %52 = load i32, i32* %6, align 4
  %53 = add nsw i32 %52, 1
  %54 = sext i32 %53 to i64
  %55 = getelementptr inbounds i32, i32* %51, i64 %54
  store i32 %50, i32* %55, align 4
  br label %56

; <label>:56:                                     ; preds = %34, %21
  br label %57

; <label>:57:                                     ; preds = %56
  %58 = load i32, i32* %6, align 4
  %59 = add nsw i32 %58, 1
  store i32 %59, i32* %6, align 4
  br label %14

; <label>:60:                                     ; preds = %14
  br label %61

; <label>:61:                                     ; preds = %60
  %62 = load i32, i32* %5, align 4
  %63 = add nsw i32 %62, 1
  store i32 %63, i32* %5, align 4
  br label %8

; <label>:64:                                     ; preds = %8
  ret void
}
```
