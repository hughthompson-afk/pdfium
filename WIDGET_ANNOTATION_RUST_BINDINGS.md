# Adding Widget Annotation Creation Functions to pdfium-render Rust Bindings

This guide explains how to integrate the newly implemented widget annotation functions into your Rust bindings for pdfium-render.

## Overview of New Functions

### Document-Level Functions
- `FPDF_EnsureAcroForm()` - Ensures the document has an AcroForm dictionary

### Widget Creation
- `FPDFPage_CreateWidgetAnnot()` - Create widget annotations with optional initialization

### Form Field Options (Choice Fields)
- `FPDFAnnot_SetFormFieldOptionArray()` - Set options for combo/list boxes
- `FPDFAnnot_SetFormFieldOptionArrayWithExportValues()` - Set options with export values

### Text Field Properties
- `FPDFAnnot_SetFormFieldMaxLen()` / `FPDFAnnot_GetFormFieldMaxLen()` - Max text length
- `FPDFAnnot_SetFormFieldQuadding()` / `FPDFAnnot_GetFormFieldQuadding()` - Text alignment

### Default Appearance & Value
- `FPDFAnnot_SetFormFieldDefaultAppearance()` / `FPDFAnnot_GetFormFieldDefaultAppearance()`
- `FPDFAnnot_SetFormFieldDefaultValue()` / `FPDFAnnot_GetFormFieldDefaultValue()`

### Button Appearance (MK Dictionary)
- `FPDFAnnot_SetFormFieldMKNormalCaption()` / `FPDFAnnot_GetFormFieldMKNormalCaption()`
- `FPDFAnnot_SetFormFieldMKRolloverCaption()` / `FPDFAnnot_GetFormFieldMKRolloverCaption()`
- `FPDFAnnot_SetFormFieldMKDownCaption()` / `FPDFAnnot_GetFormFieldMKDownCaption()`
- `FPDFAnnot_SetFormFieldMKBorderColor()` / `FPDFAnnot_GetFormFieldMKBorderColor()`
- `FPDFAnnot_SetFormFieldMKBackgroundColor()` / `FPDFAnnot_GetFormFieldMKBackgroundColor()`
- `FPDFAnnot_SetFormFieldMKRotation()` / `FPDFAnnot_GetFormFieldMKRotation()`
- `FPDFAnnot_SetFormFieldMKTextPosition()` / `FPDFAnnot_GetFormFieldMKTextPosition()`
- `FPDFAnnot_SetFormFieldMKIconFit()` / `FPDFAnnot_GetFormFieldMKIconFit()`

---

## C API Signatures

### Core Functions

```c
// Ensures the document has an /AcroForm dictionary in its catalog.
FPDF_BOOL FPDF_EnsureAcroForm(FPDF_DOCUMENT document);

// Create a widget annotation with optional properties.
FPDF_ANNOTATION FPDFPage_CreateWidgetAnnot(
    FPDF_PAGE page,
    FPDF_FORMHANDLE form_handle,
    FPDF_BYTESTRING field_name,
    FPDF_BYTESTRING field_type,
    const FS_RECTF* rect,
    int field_flags,                        // For /Ff: Use FPDF_FORMFLAG_* constants
                                            // For buttons: FPDF_FORMFLAG_BTN_RADIO for radio,
                                            // FPDF_FORMFLAG_BTN_PUSHBUTTON for push button,
                                            // 0 for checkbox
    const FPDF_WCHAR* const* options,      // For /Opt (choice fields), NULL to skip
    size_t option_count,                    // Number of options
    int max_length,                         // For /MaxLen (-1 to skip)
    int quadding,                           // For /Q (-1 to skip): 0=left, 1=center, 2=right
    FPDF_BYTESTRING default_appearance,    // For /DA (NULL to skip)
    FPDF_WIDESTRING default_value);        // For /DV (NULL to skip)
```

### Option Array Functions

```c
// Set options for combo box or list box (simple string format)
FPDF_BOOL FPDFAnnot_SetFormFieldOptionArray(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    const FPDF_WCHAR* const* options,
    size_t option_count);

// Set options with separate export values and display labels
FPDF_BOOL FPDFAnnot_SetFormFieldOptionArrayWithExportValues(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    const FPDF_WCHAR* const* export_values,
    const FPDF_WCHAR* const* display_labels,
    size_t option_count);
```

### MaxLen Functions

```c
FPDF_BOOL FPDFAnnot_SetFormFieldMaxLen(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    int max_length);  // 0 or negative for unlimited

int FPDFAnnot_GetFormFieldMaxLen(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot);  // Returns -1 on error, 0 for unlimited
```

### Quadding (Alignment) Functions

```c
FPDF_BOOL FPDFAnnot_SetFormFieldQuadding(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    int quadding);  // 0=left, 1=center, 2=right

int FPDFAnnot_GetFormFieldQuadding(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot);  // Returns -1 on error
```

### Default Appearance Functions

```c
FPDF_BOOL FPDFAnnot_SetFormFieldDefaultAppearance(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    FPDF_BYTESTRING da_string);  // Format: "/FontName Size Tf R G B rg"

unsigned long FPDFAnnot_GetFormFieldDefaultAppearance(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    char* buffer,
    unsigned long buflen);
```

### Default Value Functions

```c
FPDF_BOOL FPDFAnnot_SetFormFieldDefaultValue(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    FPDF_WIDESTRING value);

unsigned long FPDFAnnot_GetFormFieldDefaultValue(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    FPDF_WCHAR* buffer,
    unsigned long buflen);
```

### MK Caption Functions (Button Fields)

```c
// Normal caption (/MK/CA)
FPDF_BOOL FPDFAnnot_SetFormFieldMKNormalCaption(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    FPDF_WIDESTRING caption);

unsigned long FPDFAnnot_GetFormFieldMKNormalCaption(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    FPDF_WCHAR* buffer,
    unsigned long buflen);

// Rollover caption (/MK/RC)
FPDF_BOOL FPDFAnnot_SetFormFieldMKRolloverCaption(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    FPDF_WIDESTRING caption);

unsigned long FPDFAnnot_GetFormFieldMKRolloverCaption(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    FPDF_WCHAR* buffer,
    unsigned long buflen);

// Down caption (/MK/AC)
FPDF_BOOL FPDFAnnot_SetFormFieldMKDownCaption(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    FPDF_WIDESTRING caption);

unsigned long FPDFAnnot_GetFormFieldMKDownCaption(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    FPDF_WCHAR* buffer,
    unsigned long buflen);
```

### MK Color Functions (Button Fields)

```c
// Border color (/MK/BC)
// color_type: 0=transparent, 1=Gray, 3=RGB, 4=CMYK
FPDF_BOOL FPDFAnnot_SetFormFieldMKBorderColor(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    int color_type,
    const float* components,
    size_t component_count);

FPDF_BOOL FPDFAnnot_GetFormFieldMKBorderColor(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    int* color_type,
    float* components,
    size_t component_count);

// Background color (/MK/BG)
FPDF_BOOL FPDFAnnot_SetFormFieldMKBackgroundColor(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    int color_type,
    const float* components,
    size_t component_count);

FPDF_BOOL FPDFAnnot_GetFormFieldMKBackgroundColor(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    int* color_type,
    float* components,
    size_t component_count);
```

### MK Layout Functions (Button Fields)

```c
// Rotation (/MK/R) - 0, 90, 180, or 270 degrees
FPDF_BOOL FPDFAnnot_SetFormFieldMKRotation(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    int rotation);

int FPDFAnnot_GetFormFieldMKRotation(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot);

// Text position (/MK/TP) - 0-6
// 0=caption only, 1=icon only, 2=caption below, 3=caption above,
// 4=caption right, 5=caption left, 6=caption overlaid
FPDF_BOOL FPDFAnnot_SetFormFieldMKTextPosition(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    int position);

int FPDFAnnot_GetFormFieldMKTextPosition(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot);
```

### MK Icon Fit Functions (Button Fields)

```c
// Icon fit (/MK/IF)
// scale_when: 0=always, 1=bigger, 2=smaller, 3=never
// scale_type: 0=anamorphic, 1=proportional
FPDF_BOOL FPDFAnnot_SetFormFieldMKIconFit(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    int scale_when,
    int scale_type,
    float left_position,    // 0.0=left, 0.5=center, 1.0=right
    float bottom_position,  // 0.0=bottom, 0.5=center, 1.0=top
    FPDF_BOOL fit_bounds);

FPDF_BOOL FPDFAnnot_GetFormFieldMKIconFit(
    FPDF_FORMHANDLE hHandle,
    FPDF_ANNOTATION annot,
    int* scale_when,
    int* scale_type,
    float* left_position,
    float* bottom_position,
    FPDF_BOOL* fit_bounds);
```

---

## Step 1: Add FFI Bindings Trait Declarations

Add these to your `PdfiumLibraryBindings` trait in `src/bindings.rs`:

```rust
// ============================================
// Widget Annotation Creation & Configuration
// ============================================

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDF_EnsureAcroForm(&self, document: FPDF_DOCUMENT) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFPage_CreateWidgetAnnot(
    &self,
    page: FPDF_PAGE,
    form_handle: FPDF_FORMHANDLE,
    field_name: FPDF_BYTESTRING,
    field_type: FPDF_BYTESTRING,
    rect: *const FS_RECTF,
    field_flags: c_int,
    options: *const *const FPDF_WCHAR,
    option_count: usize,
    max_length: c_int,
    quadding: c_int,
    default_appearance: FPDF_BYTESTRING,
    default_value: FPDF_WIDESTRING,
) -> FPDF_ANNOTATION;

// ============================================
// Option Array Functions (Choice Fields)
// ============================================

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_SetFormFieldOptionArray(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    options: *const *const FPDF_WCHAR,
    option_count: usize,
) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_SetFormFieldOptionArrayWithExportValues(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    export_values: *const *const FPDF_WCHAR,
    display_labels: *const *const FPDF_WCHAR,
    option_count: usize,
) -> FPDF_BOOL;

// ============================================
// MaxLen Functions (Text Fields)
// ============================================

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_SetFormFieldMaxLen(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    max_length: c_int,
) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_GetFormFieldMaxLen(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
) -> c_int;

// ============================================
// Quadding Functions (Text Alignment)
// ============================================

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_SetFormFieldQuadding(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    quadding: c_int,
) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_GetFormFieldQuadding(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
) -> c_int;

// ============================================
// Default Appearance Functions
// ============================================

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_SetFormFieldDefaultAppearance(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    da_string: FPDF_BYTESTRING,
) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_GetFormFieldDefaultAppearance(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    buffer: *mut c_char,
    buflen: c_ulong,
) -> c_ulong;

// ============================================
// Default Value Functions
// ============================================

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_SetFormFieldDefaultValue(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    value: FPDF_WIDESTRING,
) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_GetFormFieldDefaultValue(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    buffer: *mut FPDF_WCHAR,
    buflen: c_ulong,
) -> c_ulong;

// ============================================
// MK Caption Functions (Button Fields)
// ============================================

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_SetFormFieldMKNormalCaption(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    caption: FPDF_WIDESTRING,
) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_GetFormFieldMKNormalCaption(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    buffer: *mut FPDF_WCHAR,
    buflen: c_ulong,
) -> c_ulong;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_SetFormFieldMKRolloverCaption(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    caption: FPDF_WIDESTRING,
) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_GetFormFieldMKRolloverCaption(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    buffer: *mut FPDF_WCHAR,
    buflen: c_ulong,
) -> c_ulong;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_SetFormFieldMKDownCaption(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    caption: FPDF_WIDESTRING,
) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_GetFormFieldMKDownCaption(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    buffer: *mut FPDF_WCHAR,
    buflen: c_ulong,
) -> c_ulong;

// ============================================
// MK Color Functions (Button Fields)
// ============================================

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_SetFormFieldMKBorderColor(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    color_type: c_int,
    components: *const f32,
    component_count: usize,
) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_GetFormFieldMKBorderColor(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    color_type: *mut c_int,
    components: *mut f32,
    component_count: usize,
) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_SetFormFieldMKBackgroundColor(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    color_type: c_int,
    components: *const f32,
    component_count: usize,
) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_GetFormFieldMKBackgroundColor(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    color_type: *mut c_int,
    components: *mut f32,
    component_count: usize,
) -> FPDF_BOOL;

// ============================================
// MK Layout Functions (Button Fields)
// ============================================

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_SetFormFieldMKRotation(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    rotation: c_int,
) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_GetFormFieldMKRotation(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
) -> c_int;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_SetFormFieldMKTextPosition(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    position: c_int,
) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_GetFormFieldMKTextPosition(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
) -> c_int;

// ============================================
// MK Icon Fit Functions (Button Fields)
// ============================================

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_SetFormFieldMKIconFit(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    scale_when: c_int,
    scale_type: c_int,
    left_position: f32,
    bottom_position: f32,
    fit_bounds: FPDF_BOOL,
) -> FPDF_BOOL;

#[cfg(feature = "pdfium_future")]
#[allow(non_snake_case)]
fn FPDFAnnot_GetFormFieldMKIconFit(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    scale_when: *mut c_int,
    scale_type: *mut c_int,
    left_position: *mut f32,
    bottom_position: *mut f32,
    fit_bounds: *mut FPDF_BOOL,
) -> FPDF_BOOL;
```

---

## Step 2: Implement in Binding Variants

### Static Bindings (`src/bindings/static_bindings.rs`)

```rust
#[cfg(feature = "pdfium_future")]
fn FPDFPage_CreateWidgetAnnot(
    &self,
    page: FPDF_PAGE,
    form_handle: FPDF_FORMHANDLE,
    field_name: FPDF_BYTESTRING,
    field_type: FPDF_BYTESTRING,
    rect: *const FS_RECTF,
    field_flags: c_int,
    options: *const *const FPDF_WCHAR,
    option_count: usize,
    max_length: c_int,
    quadding: c_int,
    default_appearance: FPDF_BYTESTRING,
    default_value: FPDF_WIDESTRING,
) -> FPDF_ANNOTATION {
    unsafe {
        ffi::FPDFPage_CreateWidgetAnnot(
            page, form_handle, field_name, field_type, rect,
            field_flags, options, option_count, max_length, quadding,
            default_appearance, default_value,
        )
    }
}

#[cfg(feature = "pdfium_future")]
fn FPDFAnnot_SetFormFieldOptionArray(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    options: *const *const FPDF_WCHAR,
    option_count: usize,
) -> FPDF_BOOL {
    unsafe { ffi::FPDFAnnot_SetFormFieldOptionArray(hHandle, annot, options, option_count) }
}

#[cfg(feature = "pdfium_future")]
fn FPDFAnnot_SetFormFieldMaxLen(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    max_length: c_int,
) -> FPDF_BOOL {
    unsafe { ffi::FPDFAnnot_SetFormFieldMaxLen(hHandle, annot, max_length) }
}

#[cfg(feature = "pdfium_future")]
fn FPDFAnnot_GetFormFieldMaxLen(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
) -> c_int {
    unsafe { ffi::FPDFAnnot_GetFormFieldMaxLen(hHandle, annot) }
}

// ... implement remaining functions following the same pattern ...
```

### Dynamic Bindings (`src/bindings/dynamic.rs`)

```rust
#[cfg(feature = "pdfium_future")]
fn FPDFAnnot_SetFormFieldMaxLen(
    &self,
    hHandle: FPDF_FORMHANDLE,
    annot: FPDF_ANNOTATION,
    max_length: c_int,
) -> FPDF_BOOL {
    let func: Symbol<unsafe extern "C" fn(
        FPDF_FORMHANDLE,
        FPDF_ANNOTATION,
        c_int,
    ) -> FPDF_BOOL> = unsafe {
        self.library.get(b"FPDFAnnot_SetFormFieldMaxLen\0").unwrap()
    };
    unsafe { func(hHandle, annot, max_length) }
}

// ... implement remaining functions following the same pattern ...
```

---

## Step 3: High-Level Rust API

### Enums and Types

```rust
/// Text alignment for form fields
#[derive(Debug, Clone, Copy, PartialEq, Eq, Default)]
pub enum PdfFormFieldAlignment {
    #[default]
    Left = 0,
    Center = 1,
    Right = 2,
}

/// Color type for MK dictionary colors
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum PdfFormFieldColorType {
    Transparent = 0,
    Gray = 1,
    RGB = 3,
    CMYK = 4,
}

/// MK color value
#[derive(Debug, Clone)]
pub enum PdfFormFieldColor {
    Transparent,
    Gray(f32),
    RGB(f32, f32, f32),
    CMYK(f32, f32, f32, f32),
}

/// Scale when options for icon fit
#[derive(Debug, Clone, Copy, PartialEq, Eq, Default)]
pub enum PdfIconScaleWhen {
    #[default]
    Always = 0,
    WhenBigger = 1,
    WhenSmaller = 2,
    Never = 3,
}

/// Scale type for icon fit
#[derive(Debug, Clone, Copy, PartialEq, Eq, Default)]
pub enum PdfIconScaleType {
    Anamorphic = 0,
    #[default]
    Proportional = 1,
}

/// Icon fit configuration
#[derive(Debug, Clone)]
pub struct PdfIconFit {
    pub scale_when: PdfIconScaleWhen,
    pub scale_type: PdfIconScaleType,
    pub left_position: f32,   // 0.0 = left, 0.5 = center, 1.0 = right
    pub bottom_position: f32, // 0.0 = bottom, 0.5 = center, 1.0 = top
    pub fit_bounds: bool,
}

impl Default for PdfIconFit {
    fn default() -> Self {
        Self {
            scale_when: PdfIconScaleWhen::Always,
            scale_type: PdfIconScaleType::Proportional,
            left_position: 0.5,
            bottom_position: 0.5,
            fit_bounds: false,
        }
    }
}
```

### Widget Annotation Methods

```rust
impl PdfPageWidgetAnnotation<'_> {
    // ============================================
    // MaxLen (Text Fields)
    // ============================================

    /// Sets the maximum text length for a text field.
    /// Pass 0 or negative for unlimited.
    #[cfg(feature = "pdfium_future")]
    pub fn set_max_length(&mut self, max_length: i32) -> Result<(), PdfiumError> {
        if !self.bindings().is_true(
            self.bindings().FPDFAnnot_SetFormFieldMaxLen(
                self.form_handle(),
                self.handle(),
                max_length,
            )
        ) {
            return Err(PdfiumError::PdfiumLibraryInternalError(
                PdfiumInternalError::Unknown,
            ));
        }
        Ok(())
    }

    /// Gets the maximum text length for a text field.
    /// Returns 0 for unlimited, -1 on error.
    #[cfg(feature = "pdfium_future")]
    pub fn max_length(&self) -> Option<i32> {
        let result = self.bindings().FPDFAnnot_GetFormFieldMaxLen(
            self.form_handle(),
            self.handle(),
        );
        if result < 0 { None } else { Some(result) }
    }

    // ============================================
    // Quadding (Text Alignment)
    // ============================================

    /// Sets the text alignment for a text field.
    #[cfg(feature = "pdfium_future")]
    pub fn set_alignment(&mut self, alignment: PdfFormFieldAlignment) -> Result<(), PdfiumError> {
        if !self.bindings().is_true(
            self.bindings().FPDFAnnot_SetFormFieldQuadding(
                self.form_handle(),
                self.handle(),
                alignment as c_int,
            )
        ) {
            return Err(PdfiumError::PdfiumLibraryInternalError(
                PdfiumInternalError::Unknown,
            ));
        }
        Ok(())
    }

    /// Gets the text alignment for a text field.
    #[cfg(feature = "pdfium_future")]
    pub fn alignment(&self) -> Option<PdfFormFieldAlignment> {
        match self.bindings().FPDFAnnot_GetFormFieldQuadding(
            self.form_handle(),
            self.handle(),
        ) {
            0 => Some(PdfFormFieldAlignment::Left),
            1 => Some(PdfFormFieldAlignment::Center),
            2 => Some(PdfFormFieldAlignment::Right),
            _ => None,
        }
    }

    // ============================================
    // Options (Choice Fields)
    // ============================================

    /// Sets the options for a combo box or list box.
    #[cfg(feature = "pdfium_future")]
    pub fn set_options(&mut self, options: &[&str]) -> Result<(), PdfiumError> {
        let wide_strings: Vec<Vec<u16>> = options
            .iter()
            .map(|s| {
                let mut v: Vec<u16> = s.encode_utf16().collect();
                v.push(0);
                v
            })
            .collect();
        
        let ptrs: Vec<*const FPDF_WCHAR> = wide_strings
            .iter()
            .map(|v| v.as_ptr())
            .collect();
        
        if !self.bindings().is_true(
            self.bindings().FPDFAnnot_SetFormFieldOptionArray(
                self.form_handle(),
                self.handle(),
                ptrs.as_ptr(),
                ptrs.len(),
            )
        ) {
            return Err(PdfiumError::PdfiumLibraryInternalError(
                PdfiumInternalError::Unknown,
            ));
        }
        Ok(())
    }

    // ============================================
    // Default Appearance
    // ============================================

    /// Sets the default appearance string.
    /// Format: "/FontName Size Tf R G B rg"
    /// Example: "/Helv 12 Tf 0 0 0 rg"
    #[cfg(feature = "pdfium_future")]
    pub fn set_default_appearance(&mut self, da: &str) -> Result<(), PdfiumError> {
        let cstr = CString::new(da).map_err(|_| PdfiumError::InvalidArgument)?;
        if !self.bindings().is_true(
            self.bindings().FPDFAnnot_SetFormFieldDefaultAppearance(
                self.form_handle(),
                self.handle(),
                cstr.as_ptr(),
            )
        ) {
            return Err(PdfiumError::PdfiumLibraryInternalError(
                PdfiumInternalError::Unknown,
            ));
        }
        Ok(())
    }

    /// Gets the default appearance string.
    #[cfg(feature = "pdfium_future")]
    pub fn default_appearance(&self) -> Option<String> {
        let length = self.bindings().FPDFAnnot_GetFormFieldDefaultAppearance(
            self.form_handle(),
            self.handle(),
            std::ptr::null_mut(),
            0,
        );
        if length == 0 {
            return None;
        }
        let mut buffer = vec![0u8; length as usize];
        self.bindings().FPDFAnnot_GetFormFieldDefaultAppearance(
            self.form_handle(),
            self.handle(),
            buffer.as_mut_ptr() as *mut c_char,
            length,
        );
        buffer.pop(); // Remove null terminator
        String::from_utf8(buffer).ok()
    }

    // ============================================
    // Default Value
    // ============================================

    /// Sets the default value (used when form is reset).
    #[cfg(feature = "pdfium_future")]
    pub fn set_default_value(&mut self, value: &str) -> Result<(), PdfiumError> {
        let wide: Vec<u16> = value.encode_utf16().chain(std::iter::once(0)).collect();
        if !self.bindings().is_true(
            self.bindings().FPDFAnnot_SetFormFieldDefaultValue(
                self.form_handle(),
                self.handle(),
                wide.as_ptr(),
            )
        ) {
            return Err(PdfiumError::PdfiumLibraryInternalError(
                PdfiumInternalError::Unknown,
            ));
        }
        Ok(())
    }

    // ============================================
    // MK Caption (Button Fields)
    // ============================================

    /// Sets the normal caption for a button.
    #[cfg(feature = "pdfium_future")]
    pub fn set_caption(&mut self, caption: &str) -> Result<(), PdfiumError> {
        let wide: Vec<u16> = caption.encode_utf16().chain(std::iter::once(0)).collect();
        if !self.bindings().is_true(
            self.bindings().FPDFAnnot_SetFormFieldMKNormalCaption(
                self.form_handle(),
                self.handle(),
                wide.as_ptr(),
            )
        ) {
            return Err(PdfiumError::PdfiumLibraryInternalError(
                PdfiumInternalError::Unknown,
            ));
        }
        Ok(())
    }

    // ============================================
    // MK Rotation (Button Fields)
    // ============================================

    /// Sets the rotation for a button (0, 90, 180, or 270 degrees).
    #[cfg(feature = "pdfium_future")]
    pub fn set_rotation(&mut self, degrees: i32) -> Result<(), PdfiumError> {
        if degrees != 0 && degrees != 90 && degrees != 180 && degrees != 270 {
            return Err(PdfiumError::InvalidArgument);
        }
        if !self.bindings().is_true(
            self.bindings().FPDFAnnot_SetFormFieldMKRotation(
                self.form_handle(),
                self.handle(),
                degrees,
            )
        ) {
            return Err(PdfiumError::PdfiumLibraryInternalError(
                PdfiumInternalError::Unknown,
            ));
        }
        Ok(())
    }

    /// Gets the rotation for a button.
    #[cfg(feature = "pdfium_future")]
    pub fn rotation(&self) -> Option<i32> {
        let result = self.bindings().FPDFAnnot_GetFormFieldMKRotation(
            self.form_handle(),
            self.handle(),
        );
        if result < 0 { None } else { Some(result) }
    }

    // ============================================
    // MK Colors (Button Fields)
    // ============================================

    /// Sets the border color for a button.
    #[cfg(feature = "pdfium_future")]
    pub fn set_border_color(&mut self, color: PdfFormFieldColor) -> Result<(), PdfiumError> {
        let (color_type, components): (c_int, Vec<f32>) = match color {
            PdfFormFieldColor::Transparent => (0, vec![]),
            PdfFormFieldColor::Gray(g) => (1, vec![g]),
            PdfFormFieldColor::RGB(r, g, b) => (3, vec![r, g, b]),
            PdfFormFieldColor::CMYK(c, m, y, k) => (4, vec![c, m, y, k]),
        };
        
        if !self.bindings().is_true(
            self.bindings().FPDFAnnot_SetFormFieldMKBorderColor(
                self.form_handle(),
                self.handle(),
                color_type,
                if components.is_empty() { std::ptr::null() } else { components.as_ptr() },
                components.len(),
            )
        ) {
            return Err(PdfiumError::PdfiumLibraryInternalError(
                PdfiumInternalError::Unknown,
            ));
        }
        Ok(())
    }

    /// Sets the background color for a button.
    #[cfg(feature = "pdfium_future")]
    pub fn set_background_color(&mut self, color: PdfFormFieldColor) -> Result<(), PdfiumError> {
        let (color_type, components): (c_int, Vec<f32>) = match color {
            PdfFormFieldColor::Transparent => (0, vec![]),
            PdfFormFieldColor::Gray(g) => (1, vec![g]),
            PdfFormFieldColor::RGB(r, g, b) => (3, vec![r, g, b]),
            PdfFormFieldColor::CMYK(c, m, y, k) => (4, vec![c, m, y, k]),
        };
        
        if !self.bindings().is_true(
            self.bindings().FPDFAnnot_SetFormFieldMKBackgroundColor(
                self.form_handle(),
                self.handle(),
                color_type,
                if components.is_empty() { std::ptr::null() } else { components.as_ptr() },
                components.len(),
            )
        ) {
            return Err(PdfiumError::PdfiumLibraryInternalError(
                PdfiumInternalError::Unknown,
            ));
        }
        Ok(())
    }

    // ============================================
    // MK Icon Fit (Button Fields)
    // ============================================

    /// Sets the icon fit parameters for a button.
    #[cfg(feature = "pdfium_future")]
    pub fn set_icon_fit(&mut self, icon_fit: &PdfIconFit) -> Result<(), PdfiumError> {
        if !self.bindings().is_true(
            self.bindings().FPDFAnnot_SetFormFieldMKIconFit(
                self.form_handle(),
                self.handle(),
                icon_fit.scale_when as c_int,
                icon_fit.scale_type as c_int,
                icon_fit.left_position,
                icon_fit.bottom_position,
                if icon_fit.fit_bounds { 1 } else { 0 },
            )
        ) {
            return Err(PdfiumError::PdfiumLibraryInternalError(
                PdfiumInternalError::Unknown,
            ));
        }
        Ok(())
    }
}
```

---

## Step 4: Widget Creation with Options

### Extended Widget Creation

```rust
/// Options for creating a widget annotation
#[derive(Debug, Clone, Default)]
pub struct WidgetCreationOptions<'a> {
    /// Field flags (/Ff key). Use FPDF_FORMFLAG_* constants.
    /// For buttons: FPDF_FORMFLAG_BTN_RADIO for radio buttons,
    /// FPDF_FORMFLAG_BTN_PUSHBUTTON for push buttons, 0 for checkboxes.
    pub field_flags: i32,
    /// Options for choice fields (combo/list box)
    pub options: Option<&'a [&'a str]>,
    /// Maximum text length for text fields
    pub max_length: Option<i32>,
    /// Text alignment
    pub alignment: Option<PdfFormFieldAlignment>,
    /// Default appearance string
    pub default_appearance: Option<&'a str>,
    /// Default value
    pub default_value: Option<&'a str>,
}

impl PdfPage {
    /// Creates a widget annotation with additional options.
    #[cfg(feature = "pdfium_future")]
    pub fn create_widget_annotation_with_options(
        &mut self,
        form_handle: FPDF_FORMHANDLE,
        field_name: &str,
        field_type: PdfWidgetFieldType,
        rect: PdfRect,
        options: WidgetCreationOptions,
    ) -> Result<PdfPageWidgetAnnotation, PdfiumError> {
        // Convert options to C format
        let wide_options: Option<Vec<Vec<u16>>> = options.options.map(|opts| {
            opts.iter()
                .map(|s| s.encode_utf16().chain(std::iter::once(0)).collect())
                .collect()
        });
        
        let option_ptrs: Option<Vec<*const FPDF_WCHAR>> = wide_options.as_ref().map(|opts| {
            opts.iter().map(|v| v.as_ptr()).collect()
        });
        
        let default_value_wide: Option<Vec<u16>> = options.default_value.map(|s| {
            s.encode_utf16().chain(std::iter::once(0)).collect()
        });
        
        let field_name_cstr = CString::new(field_name)?;
        let field_type_cstr = CString::new(field_type.as_str())?;
        let da_cstr = options.default_appearance.map(|s| CString::new(s)).transpose()?;
        
        let fs_rect = FS_RECTF {
            left: rect.left.value,
            top: rect.top.value,
            right: rect.right.value,
            bottom: rect.bottom.value,
        };
        
        let annot_handle = self.bindings().FPDFPage_CreateWidgetAnnot(
            self.handle(),
            form_handle,
            field_name_cstr.as_ptr(),
            field_type_cstr.as_ptr(),
            &fs_rect,
            options.field_flags,
            option_ptrs.as_ref().map(|v| v.as_ptr()).unwrap_or(std::ptr::null()),
            option_ptrs.as_ref().map(|v| v.len()).unwrap_or(0),
            options.max_length.unwrap_or(-1),
            options.alignment.map(|a| a as c_int).unwrap_or(-1),
            da_cstr.as_ref().map(|c| c.as_ptr()).unwrap_or(std::ptr::null()),
            default_value_wide.as_ref().map(|v| v.as_ptr()).unwrap_or(std::ptr::null()),
        );
        
        if annot_handle.is_null() {
            return Err(PdfiumError::PdfiumLibraryInternalError(
                PdfiumInternalError::Unknown,
            ));
        }
        
        Ok(PdfPageWidgetAnnotation::from_handle(annot_handle, self.bindings()))
    }
}
```

---

## Example Usage

```rust
use pdfium_render::prelude::*;

fn main() -> Result<(), PdfiumError> {
    let pdfium = Pdfium::new();
    let mut document = pdfium.create_new_pdf()?;
    document.ensure_acro_form()?;
    
    let mut page = document.pages_mut().create_page_at_start(
        PdfPagePaperSize::a4().width,
        PdfPagePaperSize::a4().height,
    )?;
    
    let form_handle = document.form_handle()?;
    
    // Create a text field with max length and center alignment
    let text_rect = PdfRect::new(100.0, 700.0, 300.0, 720.0);
    let mut text_field = page.create_widget_annotation_with_options(
        form_handle,
        "NameField",
        PdfWidgetFieldType::Text,
        text_rect,
        WidgetCreationOptions {
            max_length: Some(50),
            alignment: Some(PdfFormFieldAlignment::Center),
            default_appearance: Some("/Helv 12 Tf 0 0 0 rg"),
            default_value: Some("Enter name here"),
            ..Default::default()
        },
    )?;
    
    // Create a combo box with options
    let combo_rect = PdfRect::new(100.0, 650.0, 300.0, 670.0);
    let mut combo_field = page.create_widget_annotation_with_options(
        form_handle,
        "CountryField",
        PdfWidgetFieldType::Choice,
        combo_rect,
        WidgetCreationOptions {
            options: Some(&["United States", "Canada", "Mexico"]),
            default_appearance: Some("/Helv 10 Tf 0 0 0 rg"),
            ..Default::default()
        },
    )?;
    
    // Create a button with caption and colors
    let button_rect = PdfRect::new(100.0, 600.0, 200.0, 630.0);
    let mut button = page.create_widget_annotation(
        form_handle,
        "SubmitButton",
        PdfWidgetFieldType::Button,
        button_rect,
    )?;
    button.set_caption("Submit")?;
    button.set_background_color(PdfFormFieldColor::RGB(0.2, 0.4, 0.8))?;
    button.set_border_color(PdfFormFieldColor::RGB(0.0, 0.0, 0.5))?;
    
    // Modify existing field properties
    text_field.set_max_length(100)?;
    text_field.set_alignment(PdfFormFieldAlignment::Right)?;
    
    Ok(())
}
```

---

## Checklist

### FFI Bindings
- [ ] Add `FPDFPage_CreateWidgetAnnot` (extended signature)
- [ ] Add `FPDFAnnot_SetFormFieldOptionArray`
- [ ] Add `FPDFAnnot_SetFormFieldOptionArrayWithExportValues`
- [ ] Add `FPDFAnnot_SetFormFieldMaxLen` / `GetFormFieldMaxLen`
- [ ] Add `FPDFAnnot_SetFormFieldQuadding` / `GetFormFieldQuadding`
- [ ] Add `FPDFAnnot_SetFormFieldDefaultAppearance` / `GetFormFieldDefaultAppearance`
- [ ] Add `FPDFAnnot_SetFormFieldDefaultValue` / `GetFormFieldDefaultValue`
- [ ] Add `FPDFAnnot_SetFormFieldMKNormalCaption` / `GetFormFieldMKNormalCaption`
- [ ] Add `FPDFAnnot_SetFormFieldMKRolloverCaption` / `GetFormFieldMKRolloverCaption`
- [ ] Add `FPDFAnnot_SetFormFieldMKDownCaption` / `GetFormFieldMKDownCaption`
- [ ] Add `FPDFAnnot_SetFormFieldMKBorderColor` / `GetFormFieldMKBorderColor`
- [ ] Add `FPDFAnnot_SetFormFieldMKBackgroundColor` / `GetFormFieldMKBackgroundColor`
- [ ] Add `FPDFAnnot_SetFormFieldMKRotation` / `GetFormFieldMKRotation`
- [ ] Add `FPDFAnnot_SetFormFieldMKTextPosition` / `GetFormFieldMKTextPosition`
- [ ] Add `FPDFAnnot_SetFormFieldMKIconFit` / `GetFormFieldMKIconFit`

### Implementations
- [ ] Implement in static bindings
- [ ] Implement in dynamic bindings
- [ ] Implement in WASM bindings

### High-Level API
- [ ] Add `PdfFormFieldAlignment` enum
- [ ] Add `PdfFormFieldColor` enum
- [ ] Add `PdfIconScaleWhen` / `PdfIconScaleType` enums
- [ ] Add `PdfIconFit` struct
- [ ] Add `WidgetCreationOptions` struct
- [ ] Implement setter/getter methods on `PdfPageWidgetAnnotation`
- [ ] Add convenience widget creation method

### Testing
- [ ] Test widget creation with all optional parameters
- [ ] Test MaxLen set/get round-trip
- [ ] Test Quadding set/get round-trip
- [ ] Test option array setting
- [ ] Test MK caption functions
- [ ] Test MK color functions
- [ ] Test MK rotation
- [ ] Test Default Appearance
- [ ] Test Default Value

### Documentation
- [ ] Document all new public types
- [ ] Document all new public methods
- [ ] Add usage examples
