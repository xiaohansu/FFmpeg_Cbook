

# FFmpeg structure

## FFmpeg关键结构体之间的关系

视频播放器播放视频文件，需要经过以下几个步骤：**解协议，解封装，解码视音频，视音频同步**。

如果播放本地文件则以下几个步骤：**解封装，解码视音频，视音频同步**。

FFMPEG中结构体很多。最关键的结构体可以分成以下几类：

* 解协议（http,rtsp,rtmp,mms）

AVIOContext，URLProtocol，URLContext主要存储视音频使用的协议的类型以及状态。URLProtocol存储输入视音频使用的封装格式。每种协议都对应一个URLProtocol结构。（注意：FFMPEG中文件也被当做一种协议“file”）

* 解封装（flv,avi,rmvb,mp4）

AVFormatContext主要存储视音频封装格式中包含的信息；AVInputFormat存储输入视音频使用的封装格式。每种视音频封装格式都对应一个AVInputFormat 结构。

* 解码（h264,mpeg2,aac,mp3）

每个AVStream存储一个视频/音频流的相关数据；每个AVStream对应一个AVCodecContext，存储该视频/音频流使用解码方式的相关数据；每个AVCodecContext中对应一个AVCodec，包含该视频/音频对应的解码器。每种解码器都对应一个AVCodec结构。

* 存数据

视频的话，每个结构一般是存一帧；音频可能有好几帧， 解码前数据：AVPacket，解码后数据：AVFrame



![结构体关系图](../images/结构体关系图.jpeg)



## 结构体



* [AVFrame](#AVFrame)

* [AVPacket](#AVPacket)

* AVPicture

* [AVFormatContext](./struct/AVFormatContext)

* AVInputFormat    iformat

* AVCodecContext

* AVIOContext

* AVCodec

* AVStream

* [AVBuffer](#AVBuffer)

* [AVBufferRef](#AVBufferRef)

  

> ### AVPicture

```c
typedef struct AVPicture {
    uint8_t *data[AV_NUM_DATA_POINTERS];    ///< pointers to the image data planes
    int linesize[AV_NUM_DATA_POINTERS];     ///< number of bytes per line
} AVPicture;
```

 分析这个结构体最重要的一点就是：AVFrame和AVPicture的关系，AVPicture结构体的成员就是AVFrame结构体的两个成员，这样在一些函数中就可以直接通过AVPicture结构体指针去访问AVFrame结构体变量。可以进行**类型转换**。

相关函数

```c
avpicture_fill
```

https://www.cnblogs.com/nanqiang/p/10439011.html

https://blog.csdn.net/loveyaqin1990/article/details/21282455   待整理



 AVFrame始终指向一块内存数据，这块内存数据有可能由avcodec_decode_video（）函数内部负责去申请，然后在解码结束之后自动释 放。这也说明了为什么在yuv转rgb的时候需要自己去申请一块内存空间并将其绑定在AVFrame上，有可能因为
sws_scale（）并不会自动帮用户去申请内存空间，所以为了获取转化之后的RGB数据则需要自动去申请内存使用。



> ### <span id="AVFrame">AVFrame</span>

**code**:avcodec.h

```c
/**
 * Audio Video Frame.
 * New fields can be added to the end of AVFRAME with minor version
 * bumps. Similarly fields that are marked as to be only accessed by
 * av_opt_ptr() can be reordered. This allows 2 forks to add fields
 * without breaking compatibility with each other.
 * Removal, reordering and changes in the remaining cases require
 * a major version bump.
 * sizeof(AVFrame) must not be used outside libavcodec.
 */
typedef struct AVFrame {
#define AV_NUM_DATA_POINTERS 8
    /**
     * pointer to the picture/channel planes.
     * This might be different from the first allocated byte
     *
     * Some decoders access areas outside 0,0 - width,height, please
     * see avcodec_align_dimensions2(). Some filters and swscale can read
     * up to 16 bytes beyond the planes, if these filters are to be used,
     * then 16 extra bytes must be allocated.
     *
     * NOTE: Except for hwaccel formats, pointers not needed by the format
     * MUST be set to NULL.
     */
    uint8_t *data[AV_NUM_DATA_POINTERS];

    /**
     * For video, size in bytes of each picture line.
     * For audio, size in bytes of each plane.
     *
     * For audio, only linesize[0] may be set. For planar audio, each channel
     * plane must be the same size.
     *
     * For video the linesizes should be multiples of the CPUs alignment
     * preference, this is 16 or 32 for modern desktop CPUs.
     * Some code requires such alignment other code can be slower without
     * correct alignment, for yet other it makes no difference.
     *
     * @note The linesize may be larger than the size of usable data -- there
     * may be extra padding present for performance reasons.
     */
    int linesize[AV_NUM_DATA_POINTERS];

    /**
     * pointers to the data planes/channels.
     *
     * For video, this should simply point to data[].
     *
     * For planar audio, each channel has a separate data pointer, and
     * linesize[0] contains the size of each channel buffer.
     * For packed audio, there is just one data pointer, and linesize[0]
     * contains the total size of the buffer for all channels.
     *
     * Note: Both data and extended_data should always be set in a valid frame,
     * but for planar audio with more channels that can fit in data,
     * extended_data must be used in order to access all channels.
     */
    uint8_t **extended_data;

    /**
     * @name Video dimensions
     * Video frames only. The coded dimensions (in pixels) of the video frame,
     * i.e. the size of the rectangle that contains some well-defined values.
     *
     * @note The part of the frame intended for display/presentation is further
     * restricted by the @ref cropping "Cropping rectangle".
     * @{
     */
    int width, height;
    /**
     * @}
     */

    /**
     * number of audio samples (per channel) described by this frame
     */
    int nb_samples;

    /**
     * format of the frame, -1 if unknown or unset
     * Values correspond to enum AVPixelFormat for video frames,
     * enum AVSampleFormat for audio)
     */
    int format;

    /**
     * 1 -> keyframe, 0-> not
     */
    int key_frame;

    /**
     * Picture type of the frame.
     */
    enum AVPictureType pict_type;

    /**
     * Sample aspect ratio for the video frame, 0/1 if unknown/unspecified.
     */
    AVRational sample_aspect_ratio;

    /**
     * Presentation timestamp in time_base units (time when frame should be shown to user).
     */
    int64_t pts;

#if FF_API_PKT_PTS
    /**
     * PTS copied from the AVPacket that was decoded to produce this frame.
     * @deprecated use the pts field instead
     */
    attribute_deprecated
    int64_t pkt_pts;
#endif

    /**
     * DTS copied from the AVPacket that triggered returning this frame. (if frame threading isn't used)
     * This is also the Presentation time of this AVFrame calculated from
     * only AVPacket.dts values without pts values.
     */
    int64_t pkt_dts;

    /**
     * picture number in bitstream order
     */
    int coded_picture_number;
    /**
     * picture number in display order
     */
    int display_picture_number;

    /**
     * quality (between 1 (good) and FF_LAMBDA_MAX (bad))
     */
    int quality;

    /**
     * for some private data of the user
     */
    void *opaque;

#if FF_API_ERROR_FRAME
    /**
     * @deprecated unused
     */
    attribute_deprecated
    uint64_t error[AV_NUM_DATA_POINTERS];
#endif

    /**
     * When decoding, this signals how much the picture must be delayed.
     * extra_delay = repeat_pict / (2*fps)
     */
    int repeat_pict;

    /**
     * The content of the picture is interlaced.
     */
    int interlaced_frame;

    /**
     * If the content is interlaced, is top field displayed first.
     */
    int top_field_first;

    /**
     * Tell user application that palette has changed from previous frame.
     */
    int palette_has_changed;

    /**
     * reordered opaque 64 bits (generally an integer or a double precision float
     * PTS but can be anything).
     * The user sets AVCodecContext.reordered_opaque to represent the input at
     * that time,
     * the decoder reorders values as needed and sets AVFrame.reordered_opaque
     * to exactly one of the values provided by the user through AVCodecContext.reordered_opaque
     */
    int64_t reordered_opaque;

    /**
     * Sample rate of the audio data.
     */
    int sample_rate;

    /**
     * Channel layout of the audio data.
     */
    uint64_t channel_layout;

    /**
     * AVBuffer references backing the data for this frame. If all elements of
     * this array are NULL, then this frame is not reference counted. This array
     * must be filled contiguously -- if buf[i] is non-NULL then buf[j] must
     * also be non-NULL for all j < i.
     *
     * There may be at most one AVBuffer per data plane, so for video this array
     * always contains all the references. For planar audio with more than
     * AV_NUM_DATA_POINTERS channels, there may be more buffers than can fit in
     * this array. Then the extra AVBufferRef pointers are stored in the
     * extended_buf array.
     */
    AVBufferRef *buf[AV_NUM_DATA_POINTERS];

    /**
     * For planar audio which requires more than AV_NUM_DATA_POINTERS
     * AVBufferRef pointers, this array will hold all the references which
     * cannot fit into AVFrame.buf.
     *
     * Note that this is different from AVFrame.extended_data, which always
     * contains all the pointers. This array only contains the extra pointers,
     * which cannot fit into AVFrame.buf.
     *
     * This array is always allocated using av_malloc() by whoever constructs
     * the frame. It is freed in av_frame_unref().
     */
    AVBufferRef **extended_buf;
    /**
     * Number of elements in extended_buf.
     */
    int        nb_extended_buf;

    AVFrameSideData **side_data;
    int            nb_side_data;

/**
 * @defgroup lavu_frame_flags AV_FRAME_FLAGS
 * @ingroup lavu_frame
 * Flags describing additional frame properties.
 *
 * @{
 */

/**
 * The frame data may be corrupted, e.g. due to decoding errors.
 */
#define AV_FRAME_FLAG_CORRUPT       (1 << 0)
/**
 * A flag to mark the frames which need to be decoded, but shouldn't be output.
 */
#define AV_FRAME_FLAG_DISCARD   (1 << 2)
/**
 * @}
 */

    /**
     * Frame flags, a combination of @ref lavu_frame_flags
     */
    int flags;

    /**
     * MPEG vs JPEG YUV range.
     * - encoding: Set by user
     * - decoding: Set by libavcodec
     */
    enum AVColorRange color_range;

    enum AVColorPrimaries color_primaries;

    enum AVColorTransferCharacteristic color_trc;

    /**
     * YUV colorspace type.
     * - encoding: Set by user
     * - decoding: Set by libavcodec
     */
    enum AVColorSpace colorspace;

    enum AVChromaLocation chroma_location;

    /**
     * frame timestamp estimated using various heuristics, in stream time base
     * - encoding: unused
     * - decoding: set by libavcodec, read by user.
     */
    int64_t best_effort_timestamp;

    /**
     * reordered pos from the last AVPacket that has been input into the decoder
     * - encoding: unused
     * - decoding: Read by user.
     */
    int64_t pkt_pos;

    /**
     * duration of the corresponding packet, expressed in
     * AVStream->time_base units, 0 if unknown.
     * - encoding: unused
     * - decoding: Read by user.
     */
    int64_t pkt_duration;

    /**
     * metadata.
     * - encoding: Set by user.
     * - decoding: Set by libavcodec.
     */
    AVDictionary *metadata;

    /**
     * decode error flags of the frame, set to a combination of
     * FF_DECODE_ERROR_xxx flags if the decoder produced a frame, but there
     * were errors during the decoding.
     * - encoding: unused
     * - decoding: set by libavcodec, read by user.
     */
    int decode_error_flags;
#define FF_DECODE_ERROR_INVALID_BITSTREAM   1
#define FF_DECODE_ERROR_MISSING_REFERENCE   2
#define FF_DECODE_ERROR_CONCEALMENT_ACTIVE  4
#define FF_DECODE_ERROR_DECODE_SLICES       8

    /**
     * number of audio channels, only used for audio.
     * - encoding: unused
     * - decoding: Read by user.
     */
    int channels;

    /**
     * size of the corresponding packet containing the compressed
     * frame.
     * It is set to a negative value if unknown.
     * - encoding: unused
     * - decoding: set by libavcodec, read by user.
     */
    int pkt_size;

#if FF_API_FRAME_QP
    /**
     * QP table
     */
    attribute_deprecated
    int8_t *qscale_table;
    /**
     * QP store stride
     */
    attribute_deprecated
    int qstride;

    attribute_deprecated
    int qscale_type;

    attribute_deprecated
    AVBufferRef *qp_table_buf;
#endif
    /**
     * For hwaccel-format frames, this should be a reference to the
     * AVHWFramesContext describing the frame.
     */
    AVBufferRef *hw_frames_ctx;

    /**
     * AVBufferRef for free use by the API user. FFmpeg will never check the
     * contents of the buffer ref. FFmpeg calls av_buffer_unref() on it when
     * the frame is unreferenced. av_frame_copy_props() calls create a new
     * reference with av_buffer_ref() for the target frame's opaque_ref field.
     *
     * This is unrelated to the opaque field, although it serves a similar
     * purpose.
     */
    AVBufferRef *opaque_ref;

    /**
     * @anchor cropping
     * @name Cropping
     * Video frames only. The number of pixels to discard from the the
     * top/bottom/left/right border of the frame to obtain the sub-rectangle of
     * the frame intended for presentation.
     * @{
     */
    size_t crop_top;
    size_t crop_bottom;
    size_t crop_left;
    size_t crop_right;
    /**
     * @}
     */

    /**
     * AVBufferRef for internal use by a single libav* library.
     * Must not be used to transfer data between libraries.
     * Has to be NULL when ownership of the frame leaves the respective library.
     *
     * Code outside the FFmpeg libs should never check or change the contents of the buffer ref.
     *
     * FFmpeg calls av_buffer_unref() on it when the frame is unreferenced.
     * av_frame_copy_props() calls create a new reference with av_buffer_ref()
     * for the target frame's private_ref field.
     */
    AVBufferRef *private_ref;
} AVFrame;
/*
uint8_t *data[AV_NUM_DATA_POINTERS]：解码后原始数据（对视频来说是YUV，RGB，对音频来说是PCM）

int linesize[AV_NUM_DATA_POINTERS]：data中“一行”数据的大小。注意：未必等于图像的宽，一般大于图像的宽。

int width, height：视频帧宽和高（1920x1080,1280x720...）

int nb_samples：音频的一个AVFrame中可能包含多个音频帧，在此标记包含了几个

int format：解码后原始数据类型（YUV420，YUV422，RGB24...）

int key_frame：是否是关键帧

enum AVPictureType pict_type：帧类型（I,B,P...）

AVRational sample_aspect_ratio：宽高比（16:9，4:3...）

int64_t pts：显示时间戳

int coded_picture_number：编码帧序号

int display_picture_number：显示帧序号

int8_t *qscale_table：QP表

uint8_t *mbskip_table：跳过宏块表

int16_t (*motion_val[2])[2]：运动矢量表

uint32_t *mb_type：宏块类型表

short *dct_coeff：DCT系数，这个没有提取过

int8_t *ref_index[2]：运动估计参考帧列表（貌似H.264这种比较新的标准才会涉及到多参考帧）

int interlaced_frame：是否是隔行扫描

uint8_t motion_subsample_log2：一个宏块中的运动矢量采样个数，取log的
*/
```

AVFrame的用法：

1. AVFrame对象必须调用av_frame_alloc()在堆上分配，注意此处指的是AVFrame对象本身，AVFrame对象必须调用av_frame_free()进行销毁。
2. AVFrame中包含的数据缓冲区是
3. AVFrame通常只需分配一次，然后可以多次重用，每次重用前应调用av_frame_unref()将frame复位到原始的干净可用的状态。

下面将一些重要的成员摘录出来进行说明：

> data

```c
uint8_t *data[AV_NUM_DATA_POINTERS];
```

data是一个**指针数组**，数组的每一个元素是一个指针，指向视频中图像的某一plane或音频中某一声道的plane。

* 对于packet格式
  * 一幅YUV图像的Y、U、V交织存储在一个plane中，形如YUVYUV...，data[0]指向这个plane；
  * 一个双声道的音频帧其左声道L、右声道R交织存储在一个plane中，形如LRLRLR...，data[0]指向这个plane。

* 对于planar格式，
  * 一幅YUV图像有Y、U、V三个plane，data[0]指向Y plane，data[1]指向U plane，data[2]指向V plane；
  * 一个双声道的音频帧有左声道L和右声道R两个plane，data[0]指向L plane，data[1]指向R plane。

**YUV的存储格式**

首先，YUV按照储存方法的不同，可以分为packeted formats和planar formats；前者是YUV分量交叉排着，类似于RGB领域的hwc；后者是YUV分量分成三个数组存放，不掺和在一起，类似于RGB领域的chw。



> linesize

```c
// AV_NUM_DATA_POINTERS = 8;
int linesize[AV_NUM_DATA_POINTERS];
```

对于视频来说，linesize是每行图像的大小(字节数)。注意有对齐要求。
对于音频来说，linesize是每个plane的大小(字节数)。音频只使用linesize[0]。对于planar音频来说，每个plane的大小必须一样。
linesize可能会因性能上的考虑而填充一些额外的数据，因此linesize可能比实际对应的音视频数据尺寸要大。



> Pict_type

```c
enum AVPictureType {
    AV_PICTURE_TYPE_NONE = 0, ///< Undefined
    AV_PICTURE_TYPE_I,     ///< Intra
    AV_PICTURE_TYPE_P,     ///< Predicted
    AV_PICTURE_TYPE_B,     ///< Bi-dir predicted
    AV_PICTURE_TYPE_S,     ///< S(GMC)-VOP MPEG4
    AV_PICTURE_TYPE_SI,    ///< Switching Intra
    AV_PICTURE_TYPE_SP,    ///< Switching Predicted
    AV_PICTURE_TYPE_BI,    ///< BI type
};
```



> qscale_table

```c
int8_t *qscale_table
```

QP表指向一块内存，里面存储的是每个宏块的**QP值**。宏块的标号是从左往右，一行一行的来的。每个宏块对应1个QP。

qscale_table[0]就是第1行第1列宏块的QP值；qscale_table[1]就是第1行第2列宏块的QP值；qscale_table[2]就是第1行第3列宏块的QP值。以此类推...

宏块的个数用下式计算：

注：宏块大小是16x16的。

每行宏块数：



```cpp
int mb_stride = pCodecCtx->width/16+1
```

宏块的总数：

```cpp
int mb_sum = ((pCodecCtx->height+15)>>4)*(pCodecCtx->width/16+1)
```



> mb_type

```c
uint32_t *mb_type
```

宏块类型表存储了一帧视频中的所有宏块的类型。其存储方式和QP表差不多。只不过其是uint32类型的，而QP表是uint8类型的。每个宏块对应一个宏块类型变量。

宏块类型如下定义所示：

```c
//The following defines may change, don't expect compatibility if you use them.
#define MB_TYPE_INTRA4x4   0x0001
#define MB_TYPE_INTRA16x16 0x0002 //FIXME H.264-specific
#define MB_TYPE_INTRA_PCM  0x0004 //FIXME H.264-specific
#define MB_TYPE_16x16      0x0008
#define MB_TYPE_16x8       0x0010
#define MB_TYPE_8x16       0x0020
#define MB_TYPE_8x8        0x0040
#define MB_TYPE_INTERLACED 0x0080
#define MB_TYPE_DIRECT2    0x0100 //FIXME
#define MB_TYPE_ACPRED     0x0200
#define MB_TYPE_GMC        0x0400
#define MB_TYPE_SKIP       0x0800
#define MB_TYPE_P0L0       0x1000
#define MB_TYPE_P1L0       0x2000
#define MB_TYPE_P0L1       0x4000
#define MB_TYPE_P1L1       0x8000
#define MB_TYPE_L0         (MB_TYPE_P0L0 | MB_TYPE_P1L0)
#define MB_TYPE_L1         (MB_TYPE_P0L1 | MB_TYPE_P1L1)
#define MB_TYPE_L0L1       (MB_TYPE_L0   | MB_TYPE_L1)
#define MB_TYPE_QUANT      0x00010000
#define MB_TYPE_CBP        0x00020000
//Note bits 24-31 are reserved for codec specific use (h264 ref0, mpeg1 0mv, ...)
```

一个宏块如果包含上述定义中的一种或两种类型，则其对应的宏块变量的对应位会被置1。
注：一个宏块可以包含好几种类型，但是有些类型是不能重复包含的，比如说一个宏块不可能既是16x16又是8x8。



> extended_data

```c
uint8_t **extended_data;
```

增加extended_data的解释。FFmpeg音频解码后的数据是存放在AVFrame结构中的。 Packed格式，frame.data[0]或frame.extended_data[0]包含所有的音频数据中。 Planar格式，frame.data[i]或者frame.extended_data[i]表示第i个声道的数据（假设声道0是第一个）, AVFrame.data数组大小固定为8，如果声道数超过8，需要从frame.extended_data获取声道数据。



> ⚠️buf

```c
AVBufferRef *buf[AV_NUM_DATA_POINTERS];
```

此帧的数据可以由AVBufferRef管理，AVBufferRef提供[AVBuffer](#AVBuffer)引用机制。这里涉及到**缓冲区引用计数**概念：
AVBuffer是FFmpeg中很常用的一种缓冲区，缓冲区使用引用计数(reference-counted)机制。
AVBufferRef则对AVBuffer缓冲区提供了一层封装，最主要的是作引用计数处理，实现了一种安全机制。**用户不应直接访问AVBuffer，应通过AVBufferRef来访问AVBuffer，以保证安全**。
FFmpeg中很多基础的数据结构都包含了AVBufferRef成员，来间接使用AVBuffer缓冲区。
相关内容参考“[FFmpeg数据结构AVBuffer](https://www.cnblogs.com/leisure_chn/p/10399048.html)”
????帧的数据缓冲区AVBuffer就是前面的data成员，用户不应直接使用data成员，应通过buf成员间接使用data成员。那extended_data又是做什么的呢????

如果buf[]的所有元素都为NULL，则此帧不会被引用计数。必须连续填充buf[] - 如果buf[i]为非NULL，则对于所有j<i，buf[j]也必须为非NULL。
每个plane最多可以有一个AVBuffer，一个AVBufferRef指针指向一个AVBuffer，一个AVBuffer引用指的就是一个AVBufferRef指针。
对于视频来说，buf[]包含所有AVBufferRef指针。对于具有多于AV_NUM_DATA_POINTERS个声道的planar音频来说，可能buf[]存不下所有的AVBbufferRef指针，多出的AVBufferRef指针存储在extended_buf数组中。



参考链接

https://www.cnblogs.com/leisure_chn/p/10404502.html

https://blog.csdn.net/leixiaohua1020/article/details/14214577



> ### <span id="AVPacket">AVPacket</span>s

**code:**avcodec.h

```c
typedef struct AVPacket {
    /**
     * A reference to the reference-counted buffer where the packet data is
     * stored.
     * May be NULL, then the packet data is not reference-counted.
     */
    AVBufferRef *buf;
    /**
     * Presentation timestamp in AVStream->time_base units; the time at which
     * the decompressed packet will be presented to the user.
     * Can be AV_NOPTS_VALUE if it is not stored in the file.
     * pts MUST be larger or equal to dts as presentation cannot happen before
     * decompression, unless one wants to view hex dumps. Some formats misuse
     * the terms dts and pts/cts to mean something different. Such timestamps
     * must be converted to true pts/dts before they are stored in AVPacket.
     */
    int64_t pts;
    /**
     * Decompression timestamp in AVStream->time_base units; the time at which
     * the packet is decompressed.
     * Can be AV_NOPTS_VALUE if it is not stored in the file.
     */
    int64_t dts;
    uint8_t *data;
    int   size;
    int   stream_index;
    /**
     * A combination of AV_PKT_FLAG values
     */
    int   flags;
    /**
     * Additional packet data that can be provided by the container.
     * Packet can contain several types of side information.
     */
    AVPacketSideData *side_data;
    int side_data_elems;

    /**
     * Duration of this packet in AVStream->time_base units, 0 if unknown.
     * Equals next_pts - this_pts in presentation order.
     */
    int64_t duration;

    int64_t pos;                            ///< byte position in stream, -1 if unknown

#if FF_API_CONVERGENCE_DURATION
    /**
     * @deprecated Same as the duration field, but as int64_t. This was required
     * for Matroska subtitles, whose duration values could overflow when the
     * duration field was still an int.
     */
    attribute_deprecated
    int64_t convergence_duration;
#endif
} AVPacket;

/*
uint8_t *data：指向保存压缩数据的指针，这就是AVPacket的实际数据;

例如对于H.264来说。1个AVPacket的data通常对应一个NAL。

注意：在这里只是对应，而不是一模一样。他们之间有微小的差别：使用FFMPEG类库分离出多媒体文件中的H.264码流

因此在使用FFMPEG进行视音频处理的时候，常常可以将得到的AVPacket的data数据直接写成文件，从而得到视音频的码流文件。

int   size：data的大小

int64_t pts：显示时间戳

int64_t dts：解码时间戳

int   stream_index：标识该AVPacket所属的视频/音频流。
*/
```



**相关函数**

操作AVPacket的函数大约有30个，主要分为：AVPacket的创建初始化，AVPacket中的data数据管理（clone，free,copy），AVPacket中的side_data数据管理。



参考链接

https://www.jianshu.com/p/bb6d3905907e



> ### <span id="AVBuffer">AVBuffer</span>

**code:**libavutil/buffer_internal.h

```c
struct AVBuffer {
    uint8_t *data; /**< data described by this buffer */
    int      size; /**< size of data in bytes */

    /**
     *  number of existing AVBufferRef instances referring to this buffer
     */
    atomic_uint refcount;

    /**
     * a callback for freeing the data
     */
    void (*free)(void *opaque, uint8_t *data);

    /**
     * an opaque pointer, to be used by the freeing callback
     */
    void *opaque;

    /**
     * A combination of AV_BUFFER_FLAG_*
     */
    int flags;

    /**
     * A combination of BUFFER_FLAG_*
     */
    int flags_internal;
};
/*
data: 缓冲区地址
size: 缓冲区大小
refcount: 引用计数值
free: 用于释放缓冲区内存的回调函数
opaque: 提供给free回调函数的参数
flags: 缓冲区标志
*/
```



AVBuffer是FFmpeg中很常用的一种缓冲区，缓冲区使用**引用计数(reference-counted)机制**。

AVBufferRef则对AVBuffer缓冲区提供了一层封装(wrapper)，最主要的是作引用计数处理，实现了一种安全机制。<span style="border-bottom:2px dashed yellow;">**用户不应直接访问AVBuffer，应通过AVBufferRef来访问AVBuffer**</span>，以保证安全。

参考链接

https://www.cnblogs.com/leisure_chn/p/10399048.html

https://www.cnblogs.com/tocy/p/ffmpeg-libavutil-avbuffer-imp.html



> ### <span id="AVBufferRef">AVBufferRef</span>

**code:**libavutil/buffer.h

```c
/**
 * A reference to a data buffer.
 *
 * The size of this struct is not a part of the public ABI and it is not meant
 * to be allocated directly.
 */
typedef struct AVBufferRef {
    AVBuffer *buffer;
      /**
     * The data buffer. It is considered writable if and only if
     * this is the only reference to the buffer, in which case
     * av_buffer_is_writable() returns 1.
     */
    uint8_t *data;
      /**
     * Size of data in bytes.
     */
    int      size;
} AVBufferRef;
/*
buffer: AVBuffer
data: 缓冲区地址，实际等于buffer->data
size: 缓冲区大小，实际等于buffer->size
*/
```



**相关函数**

AVBufferRef可以用来管理引用计数的，AVBufferRef有两个函数：av_packet_ref() 和av_packet_unref()增加和减少引用计数的

avutil中主要提供了以下几个创建AVBuffer的接口：

```c
AVBufferRef *av_buffer_alloc(int size); 

AVBufferRef *av_buffer_allocz(int size); 

AVBufferRef *av_buffer_create(uint8_t *data, int size,void (*free)(void *opaque, uint8_t *data), void *opaque, int flags); 

int av_buffer_realloc(AVBufferRef **buf, int size);
```

demo

```c
extern "C" {
#include "libavutil/buffer.h"
}
#include <iostream>
using namespace std;
 
void av_buffer_default_free_2(void *opaque, uint8_t *data)
{
	free(data);
}
 
int main()
{
	const int size = 100;
	//需要被保护的数据data
	uint8_t *data = (uint8_t *)malloc(size);
	//用ref给保护起来
	AVBufferRef * buf = av_buffer_create(data, size + 64,
		av_buffer_default_free_2, NULL, 0);
 
	//
	//加一次引用
	AVBufferRef* br = av_buffer_ref(buf);
	printf("ref count %d,%d\n", av_buffer_get_ref_count(br), av_buffer_get_ref_count(buf));
 
	//又加一次引用
	AVBufferRef* br2 = av_buffer_ref(br);
	printf("ref count %d,%d\n", av_buffer_get_ref_count(br), av_buffer_get_ref_count(buf));
 
	//开始解引用
	av_buffer_unref(&br);
	printf("ref count %d\n",  av_buffer_get_ref_count(buf));
	av_buffer_unref(&br2);
	printf("ref count %d\n", av_buffer_get_ref_count(buf));
	av_buffer_unref(&buf);
	if (NULL == buf)
	{
		printf("buf is released\n");
	}
	return 1;
}
```



**参考链接**

