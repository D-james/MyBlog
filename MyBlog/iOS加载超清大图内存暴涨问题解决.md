加载超清大图是会引起内存爆表的问题，最近一直困扰着我。
SDWebImage在加载大图时做的不是很好，加载大图内存爆表。YYWebImage会好一点，但还是不行。
当不要求图片质量的情况下，最好是在上传图片的时候就压缩图片，如果显示的时候再压缩的话也会导致内存暴涨，压缩也是很占内存的。
而对于UIImageJPEGRepresentation压缩方法，不能降低显示内存。因为显示图片所占的内存大小只与图片的分辨率有关，与图片的大小无关。UIImageJPEGRepresentation压缩方法只能降低图片大小，分辨率不变。
压缩图片的两张方式看我写的这篇[http://www.jianshu.com/p/8150a8e7c0e4](http://www.jianshu.com/p/8150a8e7c0e4)
后来找到了苹果的一个官方建议加载大图的demo
[https://developer.apple.com/library/ios/samplecode/LargeImageDownsizing/](https://developer.apple.com/library/ios/samplecode/LargeImageDownsizing/)

我改装之后的代码，可以直接用：
```
#import "LargeImageDispose.h"

#define IPAD1_IPHONE3GS
#ifdef IPAD1_IPHONE3GS
#   define kDestImageSizeMB 60.0f // The resulting image will be (x)MB of uncompressed image data.
#   define kSourceImageTileSizeMB 20.0f // The tile size will be (x)MB of uncompressed image data.
#endif

/* These constants are suggested initial values for iPad2, and iPhone 4 */
//#define IPAD2_IPHONE4
#ifdef IPAD2_IPHONE4
#   define kDestImageSizeMB 120.0f // The resulting image will be (x)MB of uncompressed image data.
#   define kSourceImageTileSizeMB 40.0f // The tile size will be (x)MB of uncompressed image data.
#endif

/* These constants are suggested initial values for iPhone3G, iPod2 and earlier devices */
//#define IPHONE3G_IPOD2_AND_EARLIER
#ifdef IPHONE3G_IPOD2_AND_EARLIER
#   define kDestImageSizeMB 30.0f // The resulting image will be (x)MB of uncompressed image data.
#   define kSourceImageTileSizeMB 10.0f // The tile size will be (x)MB of uncompressed image data.
#endif

#define bytesPerMB 1048576.0f
#define bytesPerPixel 4.0f
#define pixelsPerMB ( bytesPerMB / bytesPerPixel )
#define destTotalPixels kDestImageSizeMB * pixelsPerMB
#define tileTotalPixels kSourceImageTileSizeMB * pixelsPerMB
#define destSeemOverlap 2.0f

@interface LargeImageDispose ()
{
    CGContextRef destContext;
}
@property (strong, nonatomic) UIImage *destImage;


@end

@implementation LargeImageDispose


-(UIImage *)downsizeLargeImage:(UIImage *)image {
    
    // create an image from the image filename constant. Note this
    // doesn't actually read any pixel information from disk, as that
    // is actually done at draw time.
    UIImage *sourceImage = image;
    if( sourceImage == nil ) NSLog(@"input image not found!");
    // get the width and height of the input image using
    // core graphics image helper functions.
    CGSize sourceResolution;
    sourceResolution.width = CGImageGetWidth(sourceImage.CGImage);
    sourceResolution.height = CGImageGetHeight(sourceImage.CGImage);
    // use the width and height to calculate the total number of pixels
    // in the input image.
    float sourceTotalPixels = sourceResolution.width * sourceResolution.height;
    // calculate the number of MB that would be required to store
    // this image uncompressed in memory.
    float sourceTotalMB = sourceTotalPixels / pixelsPerMB;
    
    NSLog(@"%.2f",sourceTotalMB);

    // determine the scale ratio to apply to the input image
    // that results in an output image of the defined size.
    // see kDestImageSizeMB, and how it relates to destTotalPixels.
    float imageScale = destTotalPixels / sourceTotalPixels;
    NSLog(@"%.2f",destTotalPixels);
    // use the image scale to calcualte the output image width, height
    CGSize destResolution;
    destResolution.width = (int)( sourceResolution.width * imageScale );
    destResolution.height = (int)( sourceResolution.height * imageScale );
    // create an offscreen bitmap context that will hold the output image
    // pixel data, as it becomes available by the downscaling routine.
    // use the RGB colorspace as this is the colorspace iOS GPU is optimized for.
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    int bytesPerRow = bytesPerPixel * destResolution.width;
    // allocate enough pixel data to hold the output image.
    void* destBitmapData = malloc( bytesPerRow * destResolution.height );
    if( destBitmapData == NULL ) NSLog(@"failed to allocate space for the output image!");
    // create the output bitmap context
    
    destContext = CGBitmapContextCreate( destBitmapData, destResolution.width, destResolution.height, 8, bytesPerRow, colorSpace, kCGImageAlphaPremultipliedLast );
    //    self.destContext = destContext;
    // remember CFTypes assign/check for NULL. NSObjects assign/check for nil.
    if( destContext == NULL ) {
        free( destBitmapData );
        NSLog(@"failed to create the output bitmap context!");
    }
    // release the color space object as its job is done
    CGColorSpaceRelease( colorSpace );
    // flip the output graphics context so that it aligns with the
    // cocoa style orientation of the input document. this is needed
    // because we used cocoa's UIImage -imageNamed to open the input file.
    CGContextTranslateCTM( destContext, 0.0f, destResolution.height );
    CGContextScaleCTM( destContext, 1.0f, -1.0f );
    // now define the size of the rectangle to be used for the
    // incremental blits from the input image to the output image.
    // we use a source tile width equal to the width of the source
    // image due to the way that iOS retrieves image data from disk.
    // iOS must decode an image from disk in full width 'bands', even
    // if current graphics context is clipped to a subrect within that
    // band. Therefore we fully utilize all of the pixel data that results
    // from a decoding opertion by achnoring our tile size to the full
    // width of the input image.
    CGRect sourceTile;
    sourceTile.size.width = sourceResolution.width;
    // the source tile height is dynamic. Since we specified the size
    // of the source tile in MB, see how many rows of pixels high it
    // can be given the input image width.
    sourceTile.size.height = (int)( tileTotalPixels / sourceTile.size.width );
    NSLog(@"source tile size: %f x %f",sourceTile.size.width, sourceTile.size.height);
    sourceTile.origin.x = 0.0f;
    // the output tile is the same proportions as the input tile, but
    // scaled to image scale.
    CGRect destTile;
    destTile.size.width = destResolution.width;
    destTile.size.height = sourceTile.size.height * imageScale;
    destTile.origin.x = 0.0f;
    NSLog(@"dest tile size: %f x %f",destTile.size.width, destTile.size.height);
    // the source seem overlap is proportionate to the destination seem overlap.
    // this is the amount of pixels to overlap each tile as we assemble the ouput image.
    float sourceSeemOverlap = (int)( ( destSeemOverlap / destResolution.height ) * sourceResolution.height );
    NSLog(@"dest seem overlap: %f, source seem overlap: %f",destSeemOverlap, sourceSeemOverlap);
    CGImageRef sourceTileImageRef;
    // calculate the number of read/write opertions required to assemble the
    // output image.
    int iterations = (int)( sourceResolution.height / sourceTile.size.height );
    // if tile height doesn't divide the image height evenly, add another iteration
    // to account for the remaining pixels.
    int remainder = (int)sourceResolution.height % (int)sourceTile.size.height;
    if( remainder ) iterations++;
    // add seem overlaps to the tiles, but save the original tile height for y coordinate calculations.
    float sourceTileHeightMinusOverlap = sourceTile.size.height;
    sourceTile.size.height += sourceSeemOverlap;
    destTile.size.height += destSeemOverlap;
    NSLog(@"beginning downsize. iterations: %d, tile height: %f, remainder height: %d", iterations, sourceTile.size.height,remainder );
    for( int y = 0; y < iterations; ++y ) {
        
        NSLog(@"iteration %d of %d",y+1,iterations);
        sourceTile.origin.y = y * sourceTileHeightMinusOverlap + sourceSeemOverlap;
        destTile.origin.y = ( destResolution.height ) - ( ( y + 1 ) * sourceTileHeightMinusOverlap * imageScale + destSeemOverlap );
        // create a reference to the source image with its context clipped to the argument rect.
        sourceTileImageRef = CGImageCreateWithImageInRect( sourceImage.CGImage, sourceTile );
        // if this is the last tile, it's size may be smaller than the source tile height.
        // adjust the dest tile size to account for that difference.
        if( y == iterations - 1 && remainder ) {
            float dify = destTile.size.height;
            destTile.size.height = CGImageGetHeight( sourceTileImageRef ) * imageScale;
            dify -= destTile.size.height;
            destTile.origin.y += dify;
        }
        // read and write a tile sized portion of pixels from the input image to the output image.
        CGContextDrawImage( destContext, destTile, sourceTileImageRef );
        /* release the source tile portion pixel data. note,
         releasing the sourceTileImageRef doesn't actually release the tile portion pixel
         data that we just drew, but the call afterward does. */
        CGImageRelease( sourceTileImageRef );
        /* while CGImageCreateWithImageInRect lazily loads just the image data defined by the argument rect,
         that data is finally decoded from disk to mem when CGContextDrawImage is called. sourceTileImageRef
         maintains internally a reference to the original image, and that original image both, houses and
         caches that portion of decoded mem. Thus the following call to release the source image. */
        
        // we reallocate the source image after the pool is drained since UIImage -imageNamed
        // returns us an autoreleased object.
        if( y < iterations - 1 ) {
            sourceImage = image;

            [self performSelectorOnMainThread:@selector(createImageFromContext) withObject:nil waitUntilDone:YES];
        }
    }
    NSLog(@"downsize complete.");
    //    [self performSelectorOnMainThread:@selector(initializeScrollView:) withObject:nil waitUntilDone:YES];
    // free the context since its job is done. destImageRef retains the pixel data now.
    CGContextRelease( destContext );
    
    return self.destImage;
}

-(void)createImageFromContext {
    // create a CGImage from the offscreen image context
    CGImageRef destImageRef = CGBitmapContextCreateImage( destContext );
    if( destImageRef == NULL ) NSLog(@"destImageRef is null.");
    // wrap a UIImage around the CGImage
    self.destImage = [UIImage imageWithCGImage:destImageRef scale:1.0f orientation:UIImageOrientationDownMirrored];
    // release ownership of the CGImage, since destImage retains ownership of the object now.
    CGImageRelease( destImageRef );
    if( self.destImage == nil ) NSLog(@"destImage is nil.");
}

@end
```
这段代码只是改变了图片的渲染方式，利用GPU 进行渲染，有效降低内存，不改变图片的质量，亲测加载24M图片so easy，只占20多内存
不能用于加载太小的图片，只能用于加载大图（大概1M以上）


最近看了这篇文章[http://www.jianshu.com/p/1c9de8dea3ea](http://www.jianshu.com/p/1c9de8dea3ea)
导致加载大图内存暴涨的原因是对大图的解压缩加载。
在SDWebImage中大图禁用解压缩，可以防止内存暴涨：
```
[[SDImageCache sharedImageCache] setShouldDecompressImages:NO];
[[SDWebImageDownloader sharedDownloader] setShouldDecompressImages:NO];
```
测试只有iPhone7，7p,SE,有用，其他机型不起作用，和系统版本没有关系