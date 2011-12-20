
Downloading images from Url is very common task in iOS apps. It is however important to limit the amount of threads being used as otherwise it gets pretty heavy on the CPU and whole device can be easily clogged up.

There is a nice example from Redth on github to use, however I had problems running it as it failed to add operation to NSOperationQueue for me.

So I rewrote it to use Task Parallels from .NET 4 and it works like a charm.

Usage
-----

    Usage as follows:
    
    public class ImageManager : IUrlImageUpdated
    {
		public delegate void ImageLoadedDelegate(string id, UIImage image);
		public event ImageLoadedDelegate ImageLoaded;

		UrlImageStore imageStore;
		
        private ImageManager()
        {
			imageStore = new UrlImageStore ("myImageStore", processImage);						            
        }
						
		private static ImageManager instance;
		
		public static ImageManager Instance
		{
			get
			{
				if (instance == null)
					instance = new ImageManager ();
				
				return instance;
			}	
		}
		
		// this is the actual entrypoint you call
		public UIImage GetImage(string imageUrl)
		{
			return imageStore.RequestImage (imageUrl, imageUrl, this);
		}
			
		public void UrlImageUpdated (string id, UIImage image)
		{
			// just propagate to upper level
			if (this.ImageLoaded != null)
				this.ImageLoaded(id, image);
		}
		
		// This handles our ProcessImageDelegate
		// just a simple way for us to be able to do whatever we want to our image
		// before it gets cached, so here's where you want to resize, etc.
		UIImage processImage(string id, UIImage image)
		{
			return image;
		}		
    }
    
    public class MyUIViewController : UIViewController
    {
	    // in some UIViewController simply request the image from the manager and register for the callback delegate to update then
	    // so lets assume we have view controller and it contains imageView instance of UIImageView
	    public override ViewDidLoad ()
	    {
			// get the image by some URL
			UIImage image = ImageManager.Instance.GetProductImage (_imageUrl);	
			
			if (image == null) // it is not available cached, so we will wait for it
			{
				// register for callback
				ImageManager.Instance.ImageLoaded += HandleImageManagerInstanceImageLoaded;
			}
			else
			{
				// it exists so show it here
				if (imageView != null)
					imageView.Image = image;
				
				if (activityIndicator != null)
					activityIndicator.StopAnimating();
			}
	    }
	    
	    void HandleImageManagerInstanceImageLoaded (string id, UIImage image)
		{
			if (id == _imageUrl)
			{
				// deregister the handler
				ImageManager.Instance.ImageLoaded -= HandleImageManagerInstanceImageLoaded;
				
				this.InvokeOnMainThread (delegate {
					
					if (_imageView != null)
						_imageView.Image = image;
					
					if (_activityIndicator != null)
						_activityIndicator.StopAnimating();
				});
			}
		}
	}
