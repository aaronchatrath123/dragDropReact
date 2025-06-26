# dragDropReact

```tsx
import { Box, Typography, useTheme, IconButton, Dialog, DialogTitle, 
  DialogContent, DialogActions, Button, Fade, Tooltip } from '@mui/material';
import DeleteIcon from '@mui/icons-material/Delete';
import CloseIcon from '@mui/icons-material/Close';
import type { DragEvent } from 'react';
import { useState, useRef, useEffect, useCallback } from 'react';
import type { Layout } from 'react-grid-layout';
import GridLayout from 'react-grid-layout';
import 'react-grid-layout/css/styles.css';
import 'react-resizable/css/styles.css';
import BarChartWidget from './widgets/BarChartWidget';
import AreaChartWidget from './widgets/AreaChartWidget';
import ScatterPlotWidget from './widgets/ScatterPlotWidget';
import PieChartWidget from './widgets/PieChartWidget';
import BubbleChartWidget from './widgets/BubbleChartWidget';
import type { WidgetType } from '../App';

interface Widget {
  i: string;
  id: string;
  type: WidgetType;
  x: number;
  y: number;
  w: number;
  h: number;
}

interface DisplayProps {
  onWidgetDropped: (widgetType: WidgetType) => void;
}

// Define default sizes for different widget types
const WIDGET_DEFAULTS: Record<WidgetType, { w: number; h: number }> = {
  AreaChart: { w: 6, h: 3 },  // Half width
  ScatterPlot: { w: 6, h: 3 }, // Half width
  PieChart: { w: 4, h: 4 },   // One-third width
  BubbleChart: { w: 12, h: 4 }, // Full width
  BarChart: { w: 6, h: 3 }    // Half width
};

const Display = ({ onWidgetDropped }: DisplayProps) => {
  const [widgets, setWidgets] = useState<Widget[]>([]);
  const [containerWidth, setContainerWidth] = useState(0);
  const [isDraggingOver, setIsDraggingOver] = useState(false);
  const [dragPreview, setDragPreview] = useState<{ x: number; y: number } | null>(null);
  const [deleteDialog, setDeleteDialog] = useState<{ open: boolean; widgetId: string | null }>({
    open: false,
    widgetId: null
  });
  const [hoveredWidget, setHoveredWidget] = useState<string | null>(null);
  const containerRef = useRef<HTMLDivElement>(null);
  const dragThrottleRef = useRef<number | null>(null);
  const theme = useTheme();

  useEffect(() => {
    const updateWidth = () => {
      if (containerRef.current) {
        setContainerWidth(containerRef.current.offsetWidth - 32);
      }
    };

    updateWidth();
    const resizeObserver = new ResizeObserver(updateWidth);
    if (containerRef.current) {
      resizeObserver.observe(containerRef.current);
    }

    window.addEventListener('resize', updateWidth);
    return () => {
      window.removeEventListener('resize', updateWidth);
      resizeObserver.disconnect();
    };
  }, []);

  // Calculate optimal layout based on number of widgets and their types
  const calculateOptimalLayout = (widgets: Widget[]): Widget[] => {
    if (widgets.length === 0) return [];
    
    // Clone widgets to avoid mutating the original array
    const updatedWidgets = [...widgets];
    
    // Special layouts for different numbers of widgets
    switch (widgets.length) {
      case 1:
        // Single widget takes full width
        updatedWidgets[0] = {
          ...updatedWidgets[0],
          w: 12,
          h: 4,
          x: 0,
          y: 0
        };
        break;
        
      case 2:
        // Two widgets side by side
        updatedWidgets[0] = {
          ...updatedWidgets[0],
          w: 6,
          h: 4,
          x: 0,
          y: 0
        };
        updatedWidgets[1] = {
          ...updatedWidgets[1],
          w: 6,
          h: 4,
          x: 6,
          y: 0
        };
        break;
        
      case 3:
        // Two widgets on top, one stretched below
        updatedWidgets[0] = {
          ...updatedWidgets[0],
          w: 6,
          h: 3,
          x: 0,
          y: 0
        };
        updatedWidgets[1] = {
          ...updatedWidgets[1],
          w: 6,
          h: 3,
          x: 6,
          y: 0
        };
        updatedWidgets[2] = {
          ...updatedWidgets[2],
          w: 12,
          h: 4,
          x: 0,
          y: 3
        };
        break;
        
      default:
        // For more than 3 widgets, create a more dynamic layout
        updatedWidgets.forEach((widget, index) => {
          const row = Math.floor(index / 2);
          const isEven = index % 2 === 0;
          
          if (index === widgets.length - 1 && isEven) {
            // Last widget in a row by itself - stretch it
            widget.w = 12;
            widget.x = 0;
          } else {
            // Regular widget - half width
            widget.w = 6;
            widget.x = isEven ? 0 : 6;
          }
          
          widget.h = 3;
          widget.y = row * 3;
        });
    }
    
    return updatedWidgets;
  };

  const findAvailablePosition = (widgetWidth: number, layout: Widget[]) => {
    // Position will be determined by calculateOptimalLayout
    return { x: 0, y: 0 };
  };

  const handleDrop = useCallback((e: DragEvent<HTMLDivElement>) => {
    e.preventDefault();
    e.stopPropagation();
    setIsDraggingOver(false);
    setDragPreview(null);

    if (dragThrottleRef.current) {
      window.clearTimeout(dragThrottleRef.current);
      dragThrottleRef.current = null;
    }

    const widgetType = e.dataTransfer.getData('widget');
    if (!widgetType || !(widgetType in WIDGET_DEFAULTS)) return;

    const defaultSize = WIDGET_DEFAULTS[widgetType as WidgetType];
    const position = findAvailablePosition(defaultSize.w, widgets);

    const newWidget: Widget = {
      i: `${widgetType}-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
      id: `${widgetType}-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
      type: widgetType as WidgetType,
      ...position,
      ...defaultSize
    };

    setWidgets(currentWidgets => {
      const updatedWidgets = [...currentWidgets, newWidget];
      return calculateOptimalLayout(updatedWidgets);
    });
    onWidgetDropped(widgetType as WidgetType);
  }, [widgets, onWidgetDropped]);

  const handleDragOver = (e: DragEvent<HTMLDivElement>) => {
    e.preventDefault();
    e.stopPropagation();

    // Set dragging over state only once
    if (!isDraggingOver) {
      setIsDraggingOver(true);
    }

    // Throttle preview updates
    if (dragThrottleRef.current) {
      return;
    }

    dragThrottleRef.current = window.setTimeout(() => {
      updateDragPreview(e);
      dragThrottleRef.current = null;
    }, 50); // Throttle to 50ms
  };

  const handleDragEnter = (e: DragEvent<HTMLDivElement>) => {
    e.preventDefault();
    e.stopPropagation();
    setIsDraggingOver(true);
  };

  const handleDragLeave = (e: DragEvent<HTMLDivElement>) => {
    e.preventDefault();
    e.stopPropagation();

    // Check if we're still within the container
    const rect = containerRef.current?.getBoundingClientRect();
    if (rect) {
      const { clientX, clientY } = e;
      if (
        clientX >= rect.left &&
        clientX <= rect.right &&
        clientY >= rect.top &&
        clientY <= rect.bottom
      ) {
        return;
      }
    }

    setIsDraggingOver(false);
    setDragPreview(null);

    if (dragThrottleRef.current) {
      window.clearTimeout(dragThrottleRef.current);
      dragThrottleRef.current = null;
    }
  };

  // Cleanup throttle timeout on unmount
  useEffect(() => {
    return () => {
      if (dragThrottleRef.current) {
        window.clearTimeout(dragThrottleRef.current);
      }
    };
  }, []);

  const handleDeleteClick = useCallback((widgetId: string) => {
    console.log('Attempting to delete widget:', widgetId);
    setHoveredWidget(null);
    setDeleteDialog({ open: true, widgetId });
  }, []);

  const handleDeleteConfirm = useCallback(() => {
    if (deleteDialog.widgetId) {
      console.log('Confirming deletion of widget:', deleteDialog.widgetId);
      setWidgets(currentWidgets => {
        const remainingWidgets = currentWidgets.filter(widget => widget.id !== deleteDialog.widgetId);
        // Recalculate layout for remaining widgets
        return calculateOptimalLayout(remainingWidgets);
      });
    }
    setDeleteDialog({ open: false, widgetId: null });
    setHoveredWidget(null);
  }, [deleteDialog.widgetId]);

  const handleDeleteCancel = useCallback(() => {
    setDeleteDialog({ open: false, widgetId: null });
    setHoveredWidget(null);
  }, []);

  const renderWidget = useCallback((widget: Widget) => {
    const widgetContent = (() => {
      switch (widget.type) {
        case 'AreaChart':
          return <AreaChartWidget />;
        case 'ScatterPlot':
          return <ScatterPlotWidget />;
        case 'PieChart':
          return <PieChartWidget />;
        case 'BubbleChart':
          return <BubbleChartWidget />;
        case 'BarChart':
          return <BarChartWidget />;
        default:
          const _exhaustiveCheck: never = widget.type;
          return <Typography>Unknown Widget</Typography>;
      }
    })();

    return (
      <Box
        sx={{
          width: '100%',
          height: '100%',
          overflow: 'hidden',
          borderRadius: 1,
          bgcolor: 'background.paper',
          boxShadow: 1,
          position: 'relative',
          '&:hover .widget-controls': {
            opacity: 1,
          },
        }}
        onMouseEnter={() => setHoveredWidget(widget.id)}
        onMouseLeave={() => setHoveredWidget(null)}
      >
        <Fade in={hoveredWidget === widget.id}>
          <Box
            className="widget-controls"
            sx={{
              position: 'absolute',
              top: 8,
              right: 8,
              opacity: 0,
              transition: 'opacity 0.2s ease',
              bgcolor: 'background.paper',
              borderRadius: '50%',
              boxShadow: 2,
              zIndex: 1,
            }}
          >
            <Tooltip title="Delete widget">
              <IconButton
                size="small"
                onClick={(e) => {
                  e.stopPropagation();
                  handleDeleteClick(widget.id);
                }}
                sx={{
                  '&:hover': {
                    color: 'error.main',
                  },
                }}
              >
                <DeleteIcon fontSize="small" />
              </IconButton>
            </Tooltip>
          </Box>
        </Fade>
        {widgetContent}
      </Box>
    );
  }, [hoveredWidget, handleDeleteClick, setHoveredWidget]);

  // Generate preview layout including existing widgets and preview
  const generatePreviewLayout = () => {
    if (!dragPreview || !isDraggingOver) return widgets;
    
    const previewDimensions = getWidgetDimensions(widgets.length, dragPreview);
    const previewWidget = {
      i: 'preview',
      id: 'preview',
      type: 'AreaChart' as WidgetType, // Use a default type for preview
      ...previewDimensions
    };

    // Calculate layout including preview
    const allWidgets = [...widgets, previewWidget];
    return calculateOptimalLayout(allWidgets);
  };

  // Handle layout changes from manual resizing/dragging
  const handleLayoutChange = useCallback((newLayout: Layout[]) => {
    const actualWidgets = newLayout.filter(item => item.i !== 'preview');
    setWidgets(currentWidgets => 
      currentWidgets.map(widget => {
        const layoutItem = actualWidgets.find(item => item.i === widget.i);
        if (!layoutItem) return widget;
        return {
          ...widget,
          x: layoutItem.x,
          y: layoutItem.y,
          w: layoutItem.w,
          h: layoutItem.h,
        };
      })
    );
  }, []);

  const updateDragPreview = (e: DragEvent<HTMLDivElement>) => {
    if (containerRef.current) {
      const rect = containerRef.current.getBoundingClientRect();
      const x = Math.floor((e.clientX - rect.left) / (rect.width / 12));
      const y = Math.floor((e.clientY - rect.top) / 100);

      // Only update if position has changed
      setDragPreview(prev => {
        if (!prev || prev.x !== x || prev.y !== y) {
          return { x, y };
        }
        return prev;
      });
    }
  };

  const getWidgetDimensions = (count: number, position: { x: number; y: number }) => {
    // This is now a helper function for the preview
    return {
      w: 6, // Default to half width for preview
      h: 3,
      x: position.x,
      y: position.y
    };
  };

  return (
    <>
      <Box
        ref={containerRef}
        onDrop={handleDrop}
        onDragOver={handleDragOver}
        onDragEnter={handleDragEnter}
        onDragLeave={handleDragLeave}
        sx={{
          width: '100%',
          height: '100%',
          position: 'relative',
          overflow: 'auto',
          flex: 1,
          display: 'flex',
          flexDirection: 'column',
          border: widgets.length === 0 ? `2px dashed ${theme.palette.grey[300]}` : 'none',
          borderRadius: 2,
          transition: 'all 0.2s ease',
          backgroundColor: isDraggingOver ? theme.palette.action.hover : 'transparent',
          '&:hover': {
            backgroundColor: widgets.length === 0 ? theme.palette.action.hover : 'transparent',
          },
        }}
      >
        {containerWidth > 0 && (
          <GridLayout
            className="layout"
            layout={generatePreviewLayout()}
            cols={12}
            rowHeight={100}
            width={containerWidth}
            isDraggable
            isResizable
            compactType={null}
            preventCollision={true}
            margin={[16, 16]}
            containerPadding={[16, 16]}
            onLayoutChange={handleLayoutChange}
            style={{ height: '100%' }}
          >
            {widgets.map((widget) => (
              <div 
                key={widget.i}
                style={{
                  width: '100%',
                  height: '100%',
                  padding: '1px',
                }}
              >
                {renderWidget(widget)}
              </div>
            ))}
            {isDraggingOver && dragPreview && (
              <div 
                key="preview" 
                style={{ 
                  width: '100%',
                  height: '100%',
                  backgroundColor: theme.palette.action.hover,
                  border: `2px dashed ${theme.palette.primary.main}`,
                  borderRadius: theme.shape.borderRadius,
                  display: 'flex',
                  alignItems: 'center',
                  justifyContent: 'center'
                }}
              >
                <Typography variant="body2" color="textSecondary">
                  Drop here
                </Typography>
              </div>
            )}
          </GridLayout>
        )}
        {widgets.length === 0 && (
          <Box
            sx={{
              position: 'absolute',
              top: '50%',
              left: '50%',
              transform: 'translate(-50%, -50%)',
              textAlign: 'center',
              color: theme.palette.text.secondary,
            }}
          >
            <Typography variant="h5" gutterBottom>
              Drop Widgets Here
            </Typography>
            <Typography variant="body1" color="textSecondary">
              Drag widgets from the right panel to add them to your dashboard
            </Typography>
          </Box>
        )}
      </Box>

      <Dialog
        open={deleteDialog.open}
        onClose={handleDeleteCancel}
        aria-labelledby="delete-dialog-title"
        TransitionProps={{
          onExited: () => setHoveredWidget(null)
        }}
      >
        <DialogTitle id="delete-dialog-title" sx={{ pr: 8 }}>
          Delete Widget
          <IconButton
            aria-label="close"
            onClick={handleDeleteCancel}
            sx={{
              position: 'absolute',
              right: 8,
              top: 8,
              color: 'text.secondary',
            }}
          >
            <CloseIcon />
          </IconButton>
        </DialogTitle>
        <DialogContent>
          <Typography>
            Are you sure you want to delete this widget? This action cannot be undone.
          </Typography>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleDeleteCancel} color="inherit">
            Cancel
          </Button>
          <Button 
            onClick={handleDeleteConfirm} 
            color="error" 
            variant="contained" 
            autoFocus
          >
            Delete
          </Button>
        </DialogActions>
      </Dialog>
    </>
  );
};

export default Display;
```
