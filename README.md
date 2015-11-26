# KinectPractice
namespace SkeleTest
{
    /// <summary>
    /// Interaction logic for MainWindow.xaml
    /// </summary>
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitailizeComponent();
        }

        KinectSensor sensor;
        const int SKELETON_COUNT = 6;
        Skeleton[] allSkeletons = new Skeleton[SKELETON_COUNT];
    
        private void Window_Loaded(object sender, RoutedEventArgs e)
        {
            if(KinectSensor.KinectSensors.Count > 0)
            {
                sensor = KinectSensor.KinectSensors[0];
            }
            
            if(sensor.Status == KinectStatus.Connected)
            {
                sensor.ColorStream.Enable();
                sensor.DepthStream.Enable();
                sensor.SkeletonStream.Enable();

                sensor.DepthStream.Range = DepthRange.Near;
                sensor.SkeletonStream.EnableTrackingInNearRange = true;
                sensor.SkeletonStream.TrackingMode = SkeletonTrackingMode.Seated;
            
                sensor.AllFramesReady += new EventHandler<AllFramesReadyEventArgs>(sensor_AllFramesReady);
                sensor.Start();
            }
        }

        void sensor_AllFramesReady(object sender, AllFramesReadyEventArgs e)
        {
            using(ColorImageFrame colorFrame = e.OpenColorImageFrame())
            {
                if(colorFrame == null)
                {
                    return;
                }

                byte[] pixels = new byte[colorFrame.PixelDataLength];
                colorFrame.CopyPixelDataTo(pixels);

                int stride = colorFrame.Width * 4;

                vid.Source = BitmapSource.Create(colorFrame.Width, colorFrame.Height, 96, 96, PixelFormats.Bgr32, null, pixels, stride);
            }
                
            Skeleton me = null;
            GetSkeleton(e, ref me);

            if(me == null)
            {
                return;
            }

            GetCameraPoint(me, e);
        }
    
        private void GetSkeleton(AllFramesReadyEventArgs e, ref Skeleton me)
        {
            using(SkeletonFrame skeletonFrameData = e.OpenSkeletonFrame())
            {
                if(skeletonFrameData == null)
                {
                    return;
                }
            
                skeletonFrameData.CopySkeletonDataTo(allSkeletons);

                me = (from s in allSkeletons where s.TrackingState == SkeletonTrackingState.Tracked select s).FirstorDefault();
            }
        }

        private void GetCameraPoint(Skeleton me, AllFramesReadyEventArgs e)
        {
            using(DepthImageFrame depth = e.OpenDepthImageFrame())
            {
                if(depth == null || sensor == null)
                {
                    return;
                }
            
                DepthImagePoint headDepthPoint = depth.MapFromSkeletonPoint(me.Joints[JointType.Head].Position);
        
                ColorImagePoint headColorPoint = depth.MapToColorImagePoint(headDepthPoint.X, headDepthPoint.Y, ColorImageFormat.RgbResolution640x480Fps30);

                Canvas.SetLeft(face, headColorPoint.X - face.Width / 2);
                Canvas.SetTop(face, headColorPoint.Y - face.Height / 2);
            }
        }
        
        private void Window_Closing(object sender, System.ComponentModel.CancelEventArgs e)
        {
            sensor.Stop();
        }
    }
}
