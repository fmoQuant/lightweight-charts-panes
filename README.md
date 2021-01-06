# lightweight-charts-panes

This is the change I made to include panes to lightweight-charts version 2.0


in ChartModel class:

        ChartModel.prototype.create_subplot = function (index) {
            var pane = new Pane(this._private__timeScale, this);
            if (index !== undefined) {
                this._private__panes.splice(index, 0, pane);
            }
            else {
                // adding to the end - common case
                this._private__panes.push(pane);
            }
            var actualIndex = (index === undefined) ? this._private__panes.length - 1 : index;
            // we always do autoscaling on the creation
            // if autoscale option is true, it is ok, just recalculate by invalidation mask
            // if autoscale option is false, autoscale anyway on the first draw
            // also there is a scenario when autoscale is true in constructor and false later on applyOptions
            var mask = new InvalidateMask(3 /* Full */);
            mask.invalidatePane(actualIndex, {
                level: 0 /* None */,
                autoScale: true,
            });
            this.invalidate(mask);
            this._private__panes[actualIndex].setStretchFactor(DEFAULT_STRETCH_FACTOR * 2);
            this._private__panes[actualIndex].addDataSource(this._private__watermark, true, false);
            return pane;
        };
        ChartModel.prototype.createSeries_p = function (seriesType, options, index) {
            if (index == undefined) {
                index = 0;
            }
            var pane = this._private__panes[index];
            var series = this._private__createSeries(options, seriesType, pane);
            this._private__serieses.push(series);
            if (this._private__serieses.length === 1) {
                // call fullUpdate to recalculate chart's parts geometry
                this.fullUpdate();
            }
            else {
                this.lightUpdate();
            }
            return series;
        };

        ChartApi.prototype.addAreaSeries_p = function (options, index) {
            if (options === void 0) { options = {}; }
            patchPriceFormat(options.priceFormat);
            var strictOptions = merge(clone(seriesOptionsDefaults), areaStyleDefaults, options);
            var series = this._private__chartWidget.model().createSeries_p('Area', strictOptions, index);
            var res = new SeriesApi(series, this);
            this._private__seriesMap.set(res, series);
            this._private__seriesMapReversed.set(series, res);
            return res;
        };
        ChartApi.prototype.addBarSeries_p = function (options, index) {
            if (options === void 0) { options = {}; }
            patchPriceFormat(options.priceFormat);
            var strictOptions = merge(clone(seriesOptionsDefaults), barStyleDefaults, options);
            var series = this._private__chartWidget.model().createSeries_('Bar', strictOptions, index);
            var res = new SeriesApi(series, this);
            this._private__seriesMap.set(res, series);
            this._private__seriesMapReversed.set(series, res);
            return res;
        };
        ChartApi.prototype.addCandlestickSeries_p = function (options, index) {
            if (options === void 0) { options = {}; }
            fillUpDownCandlesticksColors(options);
            patchPriceFormat(options.priceFormat);
            var strictOptions = merge(clone(seriesOptionsDefaults), candlestickStyleDefaults, options);
            var series = this._private__chartWidget.model().createSeries_p('Candlestick', strictOptions, index);
            var res = new CandlestickSeriesApi(series, this);
            this._private__seriesMap.set(res, series);
            this._private__seriesMapReversed.set(series, res);
            return res;
        };
        ChartApi.prototype.addHistogramSeries_p = function (options, index) {
            if (options === void 0) { options = {}; }
            patchPriceFormat(options.priceFormat);
            var strictOptions = merge(clone(seriesOptionsDefaults), histogramStyleDefaults, options);
            var series = this._private__chartWidget.model().createSeries_p('Histogram', strictOptions, index);
            var res = new SeriesApi(series, this);
            this._private__seriesMap.set(res, series);
            this._private__seriesMapReversed.set(series, res);
            return res;
        };

        ChartApi.prototype.addLineSeries_p = function (options, index) {
            if (options === void 0) { options = {}; }
            patchPriceFormat(options.priceFormat);
            var strictOptions = merge(clone(seriesOptionsDefaults), lineStyleDefaults, options);
            var series = this._private__chartWidget.model().createSeries_p('Line', strictOptions, index);
            var res = new SeriesApi(series, this);
            this._private__seriesMap.set(res, series);
            this._private__seriesMapReversed.set(series, res);
            return res;
        };


Also in the standalone development change this in "syncGuiWithModel" function:

                // create and insert separator
                if (i >= 1) { // Last code i > 1 and disableResize was set to true
                    var paneSeparator = new PaneSeparator(this, i - 1, i, false);
		    .....
                }


change the i > 1 to i >= 1 and set disableResize to false
