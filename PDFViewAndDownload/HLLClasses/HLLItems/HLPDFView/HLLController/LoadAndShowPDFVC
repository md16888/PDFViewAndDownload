//
//  LoadAndShowPDFVC.m
//  PDFViewAndDownload
//
//  Created by modong on 2017/11/27.
//  Copyright © 2017年 Jeffrey hu. All rights reserved.
//

#import "LoadAndShowPDFVC.h"
#import <SVProgressHUD.h>

static const NSInteger LoadAndShowPDFVC_MaxFile = 4;

@interface LoadAndShowPDFVC ()<UIWebViewDelegate>

@property (nonatomic, strong)   UIWebView *webView;

@property (nonatomic, strong) NSString *filePath;

@property (nonatomic, strong) NSString *plistPath;

@property (nonatomic, strong) NSString *fileDirectory;

@property (nonatomic, strong) NSMutableArray *fileArray;

@property (nonatomic, strong) NSURLSessionDataTask *task;

@end

@implementation LoadAndShowPDFVC

- (void)viewDidLoad {
    [super viewDidLoad];
    
    [self setupSubViews];
    
    [self downloadPDF:self.webUrlString];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

#pragma mark - Initialize data and view
- (void)setupSubViews {
    [self.view addSubview:self.webView];
    [self.webView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.top.bottom.left.and.right.offset(0);
    }];

    [self.webView setScalesPageToFit:YES];

    [SVProgressHUD show];
}

- (UIWebView *)webView
{
    if (_webView == nil)
    {
        _webView = [[UIWebView alloc]init];
        _webView.backgroundColor = [UIColor whiteColor];
        _webView.delegate= self;
    }
    return _webView;
}

- (NSString *)filePath
{
    if (!_filePath || 0 == _filePath.length)
    {
        NSArray *urlArr = [self.webUrlString componentsSeparatedByString:@"/"];
        NSString *subPath = [NSString stringWithFormat:@"/%@", [urlArr lastObject]];
        _filePath = [self.fileDirectory stringByAppendingString:subPath];
    }
    
    return _filePath;
}

- (NSString *)plistPath
{
    if (!_plistPath || 0 == _plistPath.length)
    {
        _plistPath = [NSString stringWithFormat:@"%@/pdf.plist", self.fileDirectory];
    }
    
    return _plistPath;
}

- (NSString *)fileDirectory
{
    if (!_fileDirectory || 0 == _fileDirectory.length)
    {
        _fileDirectory = [NSString stringWithFormat:@"%@/iOS_PDF", [NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES) firstObject]];
        NSError *error;
        BOOL iscreate = [[NSFileManager defaultManager] createDirectoryAtPath:_fileDirectory withIntermediateDirectories:YES attributes:nil error:&error];
    }
    
    return _fileDirectory;
}

- (NSMutableArray *)fileArray
{
    if (!_fileArray)
    {
        _fileArray = [[NSMutableArray alloc] initWithContentsOfFile:self.plistPath];
        if (!_fileArray)
        {
            _fileArray = [[NSMutableArray alloc] init];
        }
    }
    
    return _fileArray;
}

#pragma mark - UIWebViewDelegate
- (void)webViewDidFinishLoad:(UIWebView *)webView {
    NSLog(@"加载完成");
    [SVProgressHUD dismiss];
}

- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType
{
    NSString *urlStr = [request.URL absoluteString];
    if ([urlStr containsString:@"http://guide.medlive.cn/"])
    {
        return NO;
    }
    
    return YES;
}

- (void)downloadPDF:(NSString *)downloadUrl
{
    if ([[NSFileManager defaultManager] fileExistsAtPath:self.filePath])
    {
        [self loadDocument:self.filePath];
    }
    else
    {
        [self downloadFile:downloadUrl];
    }
}

- (void)downloadFile:(NSString *)downLoadUrl
{
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0), ^{
        __weak LoadAndShowPDFVC *weakSelf = self;
        
        NSCondition *condition = [[NSCondition alloc] init];
        
        NSURLSession *session = [NSURLSession sharedSession];
        
        NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:self.webUrlString]];
        [request setHTTPMethod:@"GET"];
        [request setTimeoutInterval:15];
        
        _task = [session dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
            
            if (error) {
                [SVProgressHUD dismiss];
                NSLog(@"下载失败");
                [SVProgressHUD showInfoWithStatus:@"PDF文件下载失败"];
            }
            else {
                BOOL isWrite = [data writeToFile:self.filePath atomically:YES];
                if (isWrite)
                {
                    NSLog(@"保存成功");
                    [self writePDFFileToPlistFile];
                    dispatch_async(dispatch_get_main_queue(), ^{
                        [weakSelf loadDocument:self.filePath];
                    });
                }
                else
                {
                    NSLog(@"写入失败");
                }
            }
            
            [condition lock];
            [condition signal];
            [condition unlock];
        }];
        
        [_task resume];
        
        if (condition) {
            [condition lock];
            [condition wait];
            [condition unlock];
        }
    });

}

-(void)loadDocument:(NSString *)documentName

{
    NSURL *url = [NSURL fileURLWithPath:documentName];
    
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    
    [self.webView loadRequest:request];
}

- (void)writePDFFileToPlistFile
{
    if (self.fileArray.count < LoadAndShowPDFVC_MaxFile)
    {
        [self.fileArray insertObject:self.filePath atIndex:0];
    }
    else
    {
        NSString *deleteFile = [self.fileArray lastObject];
        if ([[NSFileManager defaultManager] fileExistsAtPath:deleteFile])
        {
            NSError *err;
            [[NSFileManager defaultManager] removeItemAtPath:deleteFile error:&err];
        }
        [self.fileArray removeObjectAtIndex:LoadAndShowPDFVC_MaxFile - 1];
        
        [self.fileArray insertObject:self.filePath atIndex:0];
    }
    
    [self.fileArray writeToFile:self.plistPath atomically:YES];
}

- (void)dealloc
{
    NSLog(@"LoadAndShowPDFVC dealloc");
}

- (void)viewDidDisappear:(BOOL)animated
{
    [super viewDidDisappear:animated];
    
    [SVProgressHUD dismiss];
}

@end
