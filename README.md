clear all
close all
movieObjInput = VideoReader('fiveCCC.wmv'); 
get(movieObjInput)     
nFrames = movieObjInput.NumberOfFrames;
for iFrame=1:nFrames
    I = read(movieObjInput,iFrame);   
    fprintf('Frame %d', iFrame), ...
    figure(1), imshow(I,[]);    
    text(20,40,sprintf('Frame %d', iFrame), ...
        'BackgroundColor', 'w', ...
        'FontSize', 10);        
    targets = findCCC(I);
    nCCC = size(targets,2);            
    for i=1:nCCC
        p = targets(:,i);
        rectangle('Position', [p(1)-4,p(2)-4,8,8], 'EdgeColor', 'r', ...
            'LineWidth', 2);
    end
    if nCCC < 5     continue;   end    
    targets = findCorrespondence(targets);  
    p = targets(:, 1);    text(p(1)+5, p(2)+5, 'UL', 'Color', 'k');
    p = targets(:, 2);    text(p(1)+5, p(2)+5, 'UM', 'Color', 'k');
    p = targets(:, 3);    text(p(1)+5, p(2)+5, 'UR', 'Color', 'k');
    p = targets(:, 4);    text(p(1)+5, p(2)+5, 'LL', 'Color', 'k');
    p = targets(:, 5);    text(p(1)+5, p(2)+5, 'LR', 'Color', 'k');
     newFrameOut = getframe;
     writeVideo(movieObjOutput,newFrameOut);
end

function targets = findCCC(I)
if size(I,3) > 1
    G = rgb2gray(I);
end
W = im2bw(G, graythresh(G));    
S = strel('disk',1, 0);
W2 = imopen(W,S);
B = ~W2;        
B2 = B;
[LW, nw] = bwlabel(W2);
[LB, nb] = bwlabel(B2);
blobsWhite = regionprops(LW, 'BoundingBox', 'Centroid', 'Area');
blobsBlack = regionprops(LB, 'BoundingBox', 'Centroid', 'Area', 'Perimeter');
nCCC = 0;      
for iw=1:nw
    for ib=1:nb
        bc = blobsBlack(ib).Centroid;
        wc = blobsWhite(iw).Centroid;
        if ~(norm(bc-wc) < 1.0)  continue; end
        if ~(blobsBlack(ib).Area > blobsWhite(iw).Area)
            continue;
        end
        nCCC = nCCC + 1;
        targets(:, nCCC) = [bc(1); bc(2)];
    end
end
return
end
